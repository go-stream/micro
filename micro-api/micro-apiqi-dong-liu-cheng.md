https://github.com/asinglestep/micro-learn/blob/master/micro/%E5%90%AF%E5%8A%A8%E6%B5%81%E7%A8%8B.md

1 micro api启动流程

main.go

	cmd.Init\(\)

	

micr/cmd/cmd.go

	func Init\(\){

		App:=cmd.App\(\) 

			=&gt;//micro/go-micro/cmd/cmd.go

			func App\(\) \*cli.App {

				return DefaultCmd.App\(\)

						=&gt;//DefaultCmd = newCmd\(\)

							=&gt;//cmd.app = cli.NewApp\(\)

								=&gt;//micro/cli/app.go 创建新的cli application

			}

		app.Commands = append\(app.Commands, api.Commands\(\)...\)

			=&gt;//micro/api/api.go

				func Commands\(\) \[\]cli.Command {

					command := cli.Command{

						Name:   "api",

						Usage:  "Run the micro API",

						Action: run,

						...

				}

		...

		cmd.Init\(\)//启动app

			=&gt;//go-micro/cmd/cmd.go

				func \(c \*cmd\) Init\(opts ...Option\) error {

					...

					c.app.RunAndExitOnError\(\)

					...

				}

			=&gt;//micro/cli/app.go

					func \(a \*App\) RunAndExitOnError\(\) {

						if err := a.Run\(os.Args\); err != nil {

										fmt.Fprintln\(os.Stderr, err\)

										os.Exit\(1\)

							}

						}

					}

					=&gt;func \(a \*App\) Run\(arguments \[\]string\) \(err error\) {

						...

						err = a.Before\(context\)//// 获取参数的值，并赋值给相关字段

						...

						args := context.Args\(\)// 未被解析命令行参数列表

						if args.Present\(\) {

							// 未被解析命令行参数列表的第一个值

							name := args.First\(\)

							// 执行command

							c := a.Command\(name\)//找到name（micro api中的api）参数对应的命令返回

								=&gt;

								func \(a \*App\) Command\(name string\)\*Command{

									for \_,c:=range a.Commands{

										//找到api命令对应的Commands，就是init时添加的api.Commands

										...

									}

									...

									

								}

							if c != nil {

								return c.Run\(context\)

									=&gt;//micro/cli/command.go

									func \(c Command\) Run\(ctx \*Content\) \(err error\){

										...

										context.Command=c

										c.Action\(context\) //就是api.Commands里面的Action:run

									}

							}

						}



						a.Action\(context\)//helpCommand.Action

						//命令不存在执行a.Action\(\), 默认执行帮助函数

						return nil

					}



2 run方法					

//micro/api/api.go

func run\(ctx \*cli.Context\) {

	...

	Handler = ctx.String\("handler"\)

	...

	// create the router

	r := mux.NewRouter\(\)

	s := &srv{r}

	var h http.Handler = s



	if ctx.GlobalBool\("enable\_stats"\) {

		st := stats.New\(\)

		r.HandleFunc\("/stats", st.StatsHandler\)

		h = st.ServeHTTP\(s\)

		st.Start\(\)

		defer st.Stop\(\)

	}



	log.Logf\("Registering RPC Handler at %s", RPCPath\)

	r.HandleFunc\(RPCPath, handler.RPC\)

		=&gt;//micro/internal/handler/rpc.go

			func RPC\(w http.ResponseWriter, r \*http.Request\) {

				// 解析请求参数，service, method, address, request

				...

				// 创建request

				req := \(\*cmd.DefaultOptions\(\).Client\).NewJsonRequest\(service, method, request\)



				// create context

				ctx := helper.RequestToContext\(r\)



				// 调用后端服务，将请求结果保存到respones中

				if len\(address\) &gt; 0 {

					err = \(\*cmd.DefaultOptions\(\).Client\).CallRemote\(ctx, address, req, &response\)

				} else {

					err = \(\*cmd.DefaultOptions\(\).Client\).Call\(ctx, req, &response\)

						=&gt;///micro/go-micro/client/rpc\_client.go

							func \(r \*rpcClient\) Call\(ctx context.Context, request Request, response interface{}, opts ...CallOption\) error {

								// next为service的调度策略函数, 调用next\(\)会返回一个service

								next, err := r.opts.Selector.Select\(request.Service\(\), callOpts.SelectOptions...\)

									=&gt;//micro/go-micro/selector/default.go

										func \(r \*defaultSelector\) Select\(service string, opts ...SelectOption\) \(Next, error\) {

											...

											// 获取service列表

											services, err := r.so.Registry.GetService\(service\)

											...

											// 返回service的调度策略函数

											return sopts.Strategy\(services\), nil

										}

								...

								调用后端服务

								...

								

							}



				}

			}

			

	switch Handler {

		

		

	}

	...

	// 创建一个server

	api := server.NewServer\(Address\)

	// 修改server.opts

	api.Init\(opts...\)

	// 注册路由

	api.Handle\("/", h\)

	// Initialise Server

	service := micro.NewService\(

		micro.Name\(Name\),

		micro.RegisterTTL\(

			time.Duration\(ctx.GlobalInt\("register\_ttl"\)\)\*time.Second,

		\),

		micro.RegisterInterval\(

			time.Duration\(ctx.GlobalInt\("register\_interval"\)\)\*time.Second,

		\),

	\)

	// Start API

	if err := api.Start\(\); err != nil {

			=&gt;//micro\internal\server\server.go

			func \(s \*server\) Start\(\) error {

				...

			}

		log.Fatal\(err\)

	}



	// Run server 阻塞

	if err := service.Run\(\); err != nil {

			=&gt;//micro\go-micro\service.go

				func \(s \*service\) Run\(\) error {

					if err := s.Start\(\); err != nil {

							=&gt;//micro\go-micro\service.go

								func \(s \*service\) Start\(\) error {

									...

									if err := s.opts.Server.Start\(\); err != nil {

										=&gt;//micro/go-micro/server/rpc\_server.go

											func \(s \*rpcServer\) Start\(\) error {

												...

											}

										return err

									}

									if err := s.opts.Server.Register\(\); err != nil {

										=&gt;//micro/go-micro/server/rpc\_server.go 注册节点信息到服务发现

											func \(s \*rpcServer\) Register\(\) error {...}

										return err

									}

									...

								}

						return err

					}

					...

				}

		log.Fatal\(err\)

	}



	// Stop API 用ctrl+c,他接受借这个信号，之后才会执行stop

	if err := api.Stop\(\); err != nil {

		log.Fatal\(err\)

	}

}





3 srv.ServeHTTP 

//httpserver启动后，访问路由，默认的处理函数就是ServeHttp

func \(s \*srv\) ServeHTTP\(w http.ResponseWriter, r \*http.Request\) {

	if origin := r.Header.Get\("Origin"\); CORS\[origin\] {

		w.Header\(\).Set\("Access-Control-Allow-Origin", origin\)

	} else if len\(origin\) &gt; 0 && CORS\["\*"\] {

		w.Header\(\).Set\("Access-Control-Allow-Origin", origin\)

	}



	w.Header\(\).Set\("Access-Control-Allow-Methods", "POST, GET, OPTIONS, PUT, DELETE"\)

	w.Header\(\).Set\("Access-Control-Allow-Headers", "Content-Type, Content-Length, Accept-Encoding, X-CSRF-Token, Authorization"\)

	w.Header\(\).Set\("Access-Control-Allow-Credentials", "true"\)



	if r.Method == "OPTIONS" {

		return

	}



	s.Router.ServeHTTP\(w, r\)//根据请求匹配处理函数

		=&gt;func \(r \*Router\) ServeHTTP\(w http.ResponseWriter, req \*http.Request\) {...}

}





4 处理方法

//micro\internal\handler\meta.go: 

metaHandler.ServeHTTP\(\), 默认处理方法，可以通过--handler={rpc、proxy、api}指定处理方法

func \(m \*metaHandler\) ServeHTTP\(w http.ResponseWriter, r \*http.Request\) {

	service, err := m.r.Route\(r\)

		=&gt;//micro/go-api/router/router.go

			func \(r \*router\) Route\(req \*http.Request\) \(\*api.Service, error\) {

				...

				switch r.opts.Handler {

					case api.Default:

						// 从本地缓存中获取endpoint

						if ep, err := r.Endpoint\(req\); err == nil {

							return ep, nil

						}



						// 将请求的路由，拆分成对应的service和method

						name, method := apiRoute\(r.opts.Namespace, req.URL.Path\)

							=&gt;////micro/go-api/router/route.go

							//apiRoute\(ns, p string\), 将请求的路由，拆分成对应的service和method，

							//eg:

							// /foo/bar/zool ==&gt; service: namespace.foo, method: Bar.Zool、

							///foo/bar ==&gt; service: namespace.foo, method: Foo.Bar、

							///v1/foo/bar ==&gt; service: namespace.v1.foo, method: Foo.Bar

						

						// 获取service

						services, err := r.c.get\(name\)

						if err != nil {

							return nil, err

						}

						...

				}

			}

	...

	switch service.Endpoint.Handler {

	// proxy handler

	case api.Proxy:

		Proxy\(nil, service, false\).ServeHTTP\(w, r\)

		=&gt;//micro/internal/handler/proxy.go

			func \(p \*proxy\) ServeHTTP\(w http.ResponseWriter, r \*http.Request\) {

				

				

			}

			

	// rpcx handler

	case api.Rpc:

		RPCX\(nil, service\).ServeHTTP\(w, r\)

	// api handler

	case api.Api:

		API\(nil, service\).ServeHTTP\(w, r\)

	// default handler: api

	default:

		API\(nil, service\).ServeHTTP\(w, r\)//根据请求的服务名获取service，发送请求到service，并返回结果

			=&gt;//micro/internal/handler/api.go

				func \(a \*apiHandler\) ServeHTTP\(w http.ResponseWriter, r \*http.Request\) {

					

					

				}

	}

	

}













































