实例1：

```go
package main

import "github.com/dgrijalva/jwt-go"
import "github.com/gorilla/context"

import "net/http"

import "fmt"
import "strings"
import "time"
//自定义请求
type MyCustomClaims struct {
    Username string `json:"username"`
    jwt.StandardClaims
}
//设置客户端cookie
func setToken(res http.ResponseWriter, req *http.Request) {
    expireToken := time.Now().Add(time.Hour * 24).Unix()
    expireCookie := time.Now().Add(time.Hour * 24)

    claims := MyCustomClaims {
        "myusername",
        jwt.StandardClaims {
            ExpiresAt: expireToken,
            Issuer: "example.com",
        },
    }

    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
		//获取token的字符串
    signedToken, _ := token.SignedString([]byte("secret"))

    cookie := http.Cookie{Name: "Auth", Value: signedToken, Expires: expireCookie, HttpOnly: true}
    http.SetCookie(res, &cookie)

    http.Redirect(res, req, "/profile", 301)
}
//验证中间件，在http请求前执行
func validate(protectedPage http.HandlerFunc) http.HandlerFunc {
    return http.HandlerFunc(func(res http.ResponseWriter, req *http.Request){
        //获取cookie
        cookie, err := req.Cookie("Auth")
        if err != nil {
            http.NotFound(res, req)
            return
        }
        //在cookie中提取token
        splitCookie := strings.Split(cookie.String(), "Auth=")
				//解析token
        token, err := jwt.ParseWithClaims(splitCookie[1], &MyCustomClaims{}, func(token *jwt.Token) (interface{}, error){
           if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok{
                return nil, fmt.Errorf("Unexpected signing method %v", token.Header["alg"])
           }     
           return []byte("secret"), nil
        }) 
       //验证token
        if claims, ok := token.Claims.(*MyCustomClaims); ok && token.Valid {
            context.Set(req, "Claims", claims)    
        } else {
            http.NotFound(res, req)
            return
        }
        
        protectedPage(res, req)
    })    
}

func profile(res http.ResponseWriter, req *http.Request){
    claims := context.Get(req, "Claims").(*MyCustomClaims)
    res.Write([]byte(claims.Username))
    context.Clear(req)
}

func homePage(res http.ResponseWriter, req *http.Request){
    res.Write([]byte("Home Page"))
}
func main(){
    http.HandleFunc("/profile", validate(profile))
    http.HandleFunc("/setToken", setToken)    
    http.HandleFunc("/", homePage)
    http.ListenAndServe(":8080", nil)
}
```

实例2：

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "strings"
    "time"

    "github.com/codegangsta/negroni"
    "github.com/dgrijalva/jwt-go"
    "github.com/dgrijalva/jwt-go/request"
)

const (
    SecretKey = "welcome to wangshubo's blog"
)

func fatal(err error) {
    if err != nil {
        log.Fatal(err)
    }
}

type UserCredentials struct {
    Username string `json:"username"`
    Password string `json:"password"`
}

type User struct {
    ID       int    `json:"id"`
    Name     string `json:"name"`
    Username string `json:"username"`
    Password string `json:"password"`
}

type Response struct {
    Data string `json:"data"`
}

type Token struct {
    Token string `json:"token"`
}


func JsonResponse(response interface{}, w http.ResponseWriter) {

    json, err := json.Marshal(response)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    w.WriteHeader(http.StatusOK)
    w.Header().Set("Content-Type", "application/json")
    w.Write(json)
}
func ProtectedHandler(w http.ResponseWriter, r *http.Request) {

    response := Response{"Gained access to protected resource"}
    JsonResponse(response, w)

}
func ValidateTokenMiddleware(w http.ResponseWriter, r *http.Request, next http.HandlerFunc) {

    token, err := request.ParseFromRequest(r, request.AuthorizationHeaderExtractor,
        func(token *jwt.Token) (interface{}, error) {
            return []byte(SecretKey), nil
        })

    if err == nil {
        if token.Valid {
            next(w, r)
        } else {
            w.WriteHeader(http.StatusUnauthorized)
            fmt.Fprint(w, "Token is not valid")
        }
    } else {
        w.WriteHeader(http.StatusUnauthorized)
        fmt.Fprint(w, "Unauthorized access to this resource")
    }

}
func LoginHandler(w http.ResponseWriter, r *http.Request) {

    var user UserCredentials

    err := json.NewDecoder(r.Body).Decode(&user)

    if err != nil {
        w.WriteHeader(http.StatusForbidden)
        fmt.Fprint(w, "Error in request")
        return
    }

    if strings.ToLower(user.Username) != "someone" {
        if user.Password != "p@ssword" {
            w.WriteHeader(http.StatusForbidden)
            fmt.Println("Error logging in")
            fmt.Fprint(w, "Invalid credentials")
            return
        }
    }

    token := jwt.New(jwt.SigningMethodHS256)
    claims := make(jwt.MapClaims)
    claims["exp"] = time.Now().Add(time.Hour * time.Duration(1)).Unix()
    claims["iat"] = time.Now().Unix()
    token.Claims = claims

    if err != nil {
        w.WriteHeader(http.StatusInternalServerError)
        fmt.Fprintln(w, "Error extracting the key")
        fatal(err)
    }

    tokenString, err := token.SignedString([]byte(SecretKey))
    if err != nil {
        w.WriteHeader(http.StatusInternalServerError)
        fmt.Fprintln(w, "Error while signing the token")
        fatal(err)
    }

    response := Token{tokenString}
    JsonResponse(response, w)

}
func StartServer() {

    http.HandleFunc("/login", LoginHandler)

    http.Handle("/resource", negroni.New(
        negroni.HandlerFunc(ValidateTokenMiddleware),
        negroni.Wrap(http.HandlerFunc(ProtectedHandler)),
    ))

    log.Println("Now listening...")
    http.ListenAndServe(":8080", nil)
}

func main() {
    StartServer()
}
```



