http.HandleFunc\("/", sayhelloName\) //设置访问的路由



net\http\server.go:

1、HandleFunc

	//为给定模式注册处理函数

	func HandleFunc\(pattern string, handler func\(ResponseWriter, \*Request\)\) {

		DefaultServeMux.HandleFunc\(pattern, handler\)

	}

2.1、DefaultServeMux

	//是Serve使用的默认ServeMux。

	var DefaultServeMux = &defaultServeMux

	var defaultServeMux ServeMux

2.2、所以DefaultServeMux.HandleFunc 就是ServeMux的 HandleFunc

	//HandleFunc为给定模式注册处理函数。

	func \(mux \*ServeMux\) HandleFunc\(pattern string, handler func\(ResponseWriter, \*Request\)\) {

		mux.Handle\(pattern, HandlerFunc\(handler\)\)

	}

2.3、 HandlerFunc

//HandlerFunc\(f\),强制类型转换f成为HandlerFunc类型，这样f就拥有了ServeHTTP方法

type HandlerFunc func\(ResponseWriter, \*Request\)

// ServeHTTP calls f\(w, r\).

func \(f HandlerFunc\) ServeHTTP\(w ResponseWriter, r \*Request\) {

	f\(w, r\)

}



3、找ServeMux的 Handle

	//Handle注册给定模式的处理程序。

	//向DefaultServeMux的map\[string\]muxEntry中增加对应的handler和路由规则

	func \(mux \*ServeMux\) Handle\(pattern string, handler Handler\) {

		mux.mu.Lock\(\)

		defer mux.mu.Unlock\(\)



		if pattern == "" {

			panic\("http: invalid pattern " + pattern\)

		}

		if handler == nil {

			panic\("http: nil handler"\)

		}

		if mux.m\[pattern\].explicit {

			panic\("http: multiple registrations for " + pattern\)

		}



		if mux.m == nil {

			mux.m = make\(map\[string\]muxEntry\)

		}

		mux.m\[pattern\] = muxEntry{explicit: true, h: handler, pattern: pattern}



		if pattern\[0\] != '/' {

			mux.hosts = true

		}



		// Helpful behavior:

		// If pattern is /tree/, insert an implicit permanent redirect for /tree.

		// It can be overridden by an explicit registration.

		n := len\(pattern\)

		if n &gt; 0 && pattern\[n-1\] == '/' && !mux.m\[pattern\[0:n-1\]\].explicit {

			// If pattern contains a host name, strip it and use remaining

			// path for redirect.

			path := pattern

			if pattern\[0\] != '/' {

				// In pattern, at least the last character is a '/', so

				// strings.Index can't be -1.

				path = pattern\[strings.Index\(pattern, "/"\):\]

			}

			url := &url.URL{Path: path}

			mux.m\[pattern\[0:n-1\]\] = muxEntry{h: RedirectHandler\(url.String\(\), StatusMovedPermanently\), pattern: pattern}

		}

	}





err := http.ListenAndServe\(":9090", nil\) //设置监听的端口

1 实例化Server

	func ListenAndServe\(addr string, handler Handler\) error {

		server := &Server{Addr: addr, Handler: handler}

		return server.ListenAndServe\(\)

	}

2 调用Server的 ListenAndServe\(\)

	func \(srv \*Server\) ListenAndServe\(\) error {

		addr := srv.Addr

		if addr == "" {

			addr = ":http"

		}

		ln, err := net.Listen\("tcp", addr\)//dial.go  

		if err != nil {

			return err

		}

		return srv.Serve\(tcpKeepAliveListener{ln.\(\*net.TCPListener\)}\)//2678

	}

3 调用 net.Listen\("tcp", addr\)监听端口

4 启动一个for循环，在循环体中 Accept 请求



	func \(srv \*Server\) Serve\(l net.Listener\) error {

		defer l.Close\(\)

		if fn := testHookServerServe; fn != nil {

			fn\(srv, l\)

		}

		var tempDelay time.Duration // how long to sleep on accept failure



		if err := srv.setupHTTP2\_Serve\(\); err != nil {

			return err

		}



		srv.trackListener\(l, true\)

		defer srv.trackListener\(l, false\)



		baseCtx := context.Background\(\) // base is always background, per Issue 16220

		ctx := context.WithValue\(baseCtx, ServerContextKey, srv\)

		for {

			rw, e := l.Accept\(\)

			if e != nil {

				select {

				case &lt;-srv.getDoneChan\(\):

					return ErrServerClosed

				default:

				}

				if ne, ok := e.\(net.Error\); ok && ne.Temporary\(\) {

					if tempDelay == 0 {

						tempDelay = 5 \* time.Millisecond

					} else {

						tempDelay \*= 2

					}

					if max := 1 \* time.Second; tempDelay &gt; max {

						tempDelay = max

					}

					srv.logf\("http: Accept error: %v; retrying in %v", e, tempDelay\)

					time.Sleep\(tempDelay\)

					continue

				}

				return e

			}

			tempDelay = 0

			c := srv.newConn\(rw\)

			c.setState\(c.rwc, StateNew\) // before Serve can return

			go c.serve\(ctx\)

		}

	}

5 对每个请求实例化一个 Conn newConn ，并且开启一个goroutine 为这个请求进行服务go c.serve\(\)

	func \(c \*conn\) setState\(nc net.Conn, state ConnState\) {

		srv := c.server

		switch state {

		case StateNew:

			srv.trackConn\(c, true\)

		case StateHijacked, StateClosed:

			srv.trackConn\(c, false\)

		}

		c.curState.Store\(connStateInterface\[state\]\)

		if hook := srv.ConnState; hook != nil {

			hook\(nc, state\)

		}

	}

6 读取每个请求的内容w, err := c.readRequest\(\)

7 判断handler是否为空，

如果没有设置handler（这个例子就没有设置handler），handler就设置为DefaultServeMux

	// Serve a new connection.

	func \(c \*conn\) serve\(ctx context.Context\) {

		c.remoteAddr = c.rwc.RemoteAddr\(\).String\(\)

		ctx = context.WithValue\(ctx, LocalAddrContextKey, c.rwc.LocalAddr\(\)\)

		defer func\(\) {

			if err := recover\(\); err != nil && err != ErrAbortHandler {

				const size = 64 &lt;&lt; 10

				buf := make\(\[\]byte, size\)

				buf = buf\[:runtime.Stack\(buf, false\)\]

				c.server.logf\("http: panic serving %v: %v\n%s", c.remoteAddr, err, buf\)

			}

			if !c.hijacked\(\) {

				c.close\(\)

				c.setState\(c.rwc, StateClosed\)

			}

		}\(\)



		if tlsConn, ok := c.rwc.\(\*tls.Conn\); ok {

			if d := c.server.ReadTimeout; d != 0 {

				c.rwc.SetReadDeadline\(time.Now\(\).Add\(d\)\)

			}

			if d := c.server.WriteTimeout; d != 0 {

				c.rwc.SetWriteDeadline\(time.Now\(\).Add\(d\)\)

			}

			if err := tlsConn.Handshake\(\); err != nil {

				c.server.logf\("http: TLS handshake error from %s: %v", c.rwc.RemoteAddr\(\), err\)

				return

			}

			c.tlsState = new\(tls.ConnectionState\)

			\*c.tlsState = tlsConn.ConnectionState\(\)

			if proto := c.tlsState.NegotiatedProtocol; validNPN\(proto\) {

				if fn := c.server.TLSNextProto\[proto\]; fn != nil {

					h := initNPNRequest{tlsConn, serverHandler{c.server}}

					fn\(c.server, tlsConn, h\)

				}

				return

			}

		}



		// HTTP/1.x from here on.



		ctx, cancelCtx := context.WithCancel\(ctx\)

		c.cancelCtx = cancelCtx

		defer cancelCtx\(\)



		c.r = &connReader{conn: c}

		c.bufr = newBufioReader\(c.r\)

		c.bufw = newBufioWriterSize\(checkConnErrorWriter{c}, 4&lt;&lt;10\)



		for {

			w, err := c.readRequest\(ctx\)

			if c.r.remain != c.server.initialReadLimitSize\(\) {

				// If we read any bytes off the wire, we're active.

				c.setState\(c.rwc, StateActive\)

			}

			if err != nil {

				const errorHeaders = "\r\nContent-Type: text/plain; charset=utf-8\r\nConnection: close\r\n\r\n"



				if err == errTooLarge {

					// Their HTTP client may or may not be

					// able to read this if we're

					// responding to them and hanging up

					// while they're still writing their

					// request. Undefined behavior.

					const publicErr = "431 Request Header Fields Too Large"

					fmt.Fprintf\(c.rwc, "HTTP/1.1 "+publicErr+errorHeaders+publicErr\)

					c.closeWriteAndWait\(\)

					return

				}

				if isCommonNetReadError\(err\) {

					return // don't reply

				}



				publicErr := "400 Bad Request"

				if v, ok := err.\(badRequestError\); ok {

					publicErr = publicErr + ": " + string\(v\)

				}



				fmt.Fprintf\(c.rwc, "HTTP/1.1 "+publicErr+errorHeaders+publicErr\)

				return

			}



			// Expect 100 Continue support

			req := w.req

			if req.expectsContinue\(\) {

				if req.ProtoAtLeast\(1, 1\) && req.ContentLength != 0 {

					// Wrap the Body reader with one that replies on the connection

					req.Body = &expectContinueReader{readCloser: req.Body, resp: w}

				}

			} else if req.Header.get\("Expect"\) != "" {

				w.sendExpectationFailed\(\)

				return

			}



			c.curReq.Store\(w\)



			if requestBodyRemains\(req.Body\) {

				registerOnHitEOF\(req.Body, w.conn.r.startBackgroundRead\)

			} else {

				if w.conn.bufr.Buffered\(\) &gt; 0 {

					w.conn.r.closeNotifyFromPipelinedRequest\(\)

				}

				w.conn.r.startBackgroundRead\(\)

			}



			// HTTP cannot have multiple simultaneous active requests.\[\*\]

			// Until the server replies to this request, it can't read another,

			// so we might as well run the handler in this goroutine.

			// \[\*\] Not strictly true: HTTP pipelining. We could let them all process

			// in parallel even if their responses need to be serialized.

			// But we're not going to implement HTTP pipelining because it

			// was never deployed in the wild and the answer is HTTP/2.

			serverHandler{c.server}.ServeHTTP\(w, w.req\)

			w.cancelCtx\(\)

			if c.hijacked\(\) {

				return

			}

			w.finishRequest\(\)

			if !w.shouldReuseConnection\(\) {

				if w.requestBodyLimitHit \|\| w.closedRequestBodyEarly\(\) {

					c.closeWriteAndWait\(\)

				}

				return

			}

			c.setState\(c.rwc, StateIdle\)

			c.curReq.Store\(\(\*response\)\(nil\)\)



			if !w.conn.server.doKeepAlives\(\) {

				// We're in shutdown mode. We might've replied

				// to the user without "Connection: close" and

				// they might think they can send another

				// request, but such is life with HTTP/1.1.

				return

			}



			if d := c.server.idleTimeout\(\); d != 0 {

				c.rwc.SetReadDeadline\(time.Now\(\).Add\(d\)\)

				if \_, err := c.bufr.Peek\(4\); err != nil {

					return

				}

			}

			c.rwc.SetReadDeadline\(time.Time{}\)

		}

	}



8 调用handler的 ServeHttp

	func \(sh serverHandler\) ServeHTTP\(rw ResponseWriter, req \*Request\) {

		handler := sh.srv.Handler

		if handler == nil {

			handler = DefaultServeMux

		}

		if req.RequestURI == "\*" && req.Method == "OPTIONS" {

			handler = globalOptionsHandler{}

		}

		handler.ServeHTTP\(rw, req\)

	}

9 在这个例子中，下面就进入到 DefaultServeMux.ServeHttp

	func \(mux \*ServeMux\) ServeHTTP\(w ResponseWriter, r \*Request\) {

		if r.RequestURI == "\*" {

			if r.ProtoAtLeast\(1, 1\) {

				w.Header\(\).Set\("Connection", "close"\)

			}

			w.WriteHeader\(StatusBadRequest\)

			return

		}

		h, \_ := mux.Handler\(r\)//h Handler // 这个路由表达式对应哪个handler

		h.ServeHTTP\(w, r\)//然后找到了对应的ServeHTTP，即自定义的函数sayHello

	}

10 根据request选择handler，

	并且进入到这个 handler 的 ServeHTTP ：mux.handler\(r\).ServeHTTP\(w, r\)

	  			//v, ok := mux.m\[path\]//muxEntry

11 选择handler：

	A 判断是否有路由能满足这个request（循环遍历ServeMux的muxEntry）

	B 如果有路由满足，调用这个路由handler的ServeHTTP

	C 如果没有路由满足，调用NotFoundHandler的ServeHTTP



	func \(mux \*ServeMux\) Handler\(r \*Request\) \(h Handler, pattern string\) {



		// CONNECT requests are not canonicalized.

		if r.Method == "CONNECT" {

			return mux.handler\(r.Host, r.URL.Path\)

		}



		// All other requests have any port stripped and path cleaned

		// before passing to mux.handler.

		host := stripHostPort\(r.Host\)

		path := cleanPath\(r.URL.Path\)

		if path != r.URL.Path {

			\_, pattern = mux.handler\(host, path\)

			url := \*r.URL

			url.Path = path

			return RedirectHandler\(url.String\(\), StatusMovedPermanently\), pattern

		}



		return mux.handler\(host, r.URL.Path\)

	}

	func \(mux \*ServeMux\) handler\(host, path string\) \(h Handler, pattern string\) {

		mux.mu.RLock\(\)

		defer mux.mu.RUnlock\(\)



		// Host-specific pattern takes precedence over generic ones

		if mux.hosts {

			h, pattern = mux.match\(host + path\)

		}

		if h == nil {

			h, pattern = mux.match\(path\)

		}

		if h == nil {

			h, pattern = NotFoundHandler\(\), ""

		}

		return

	}

	func \(mux \*ServeMux\) match\(path string\) \(h Handler, pattern string\) {

		// Check for exact match first.

		v, ok := mux.m\[path\]//muxEntry

		if ok {

			return v.h, v.pattern

		}



		// Check for longest valid match.

		var n = 0

		for k, v := range mux.m {

			if !pathMatch\(k, path\) {

				continue

			}

			if h == nil \|\| len\(k\) &gt; n {

				n = len\(k\)

				h = v.h

				pattern = v.pattern

			}

		}

		return

	}

















