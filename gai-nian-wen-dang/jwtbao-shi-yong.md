使用自定义声明类型创建令牌的示例

StandardClaim嵌入在自定义类型中，以便于标准声明的编码，解析和验证。

StandardClaims类型被设计为嵌入到自定义类型中以提供标准验证功能。

你可以单独使用它，但解析后无法检索其他字段



mySigningKey := \[\]byte\("AllYourBase"\)

type MyCustomClaims struct {

    Foo string \`json:"foo"\`

    jwt.StandardClaims

}

// Create the Claims

claims := MyCustomClaims{

    "bar",

    jwt.StandardClaims{

        ExpiresAt: 15000,

        Issuer:    "test",

    },

}

token := jwt.NewWithClaims\(jwt.SigningMethodHS256, claims\)

ss, err := token.SignedString\(mySigningKey\)

fmt.Printf\("%v %v", ss, err\)

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmb28iOiJiYXIiLCJleHAiOjE1MDAwLCJpc3MiOiJ0ZXN0In0.HE7fK0xOQwFEr4WDgRWj4teRPZ6i3GLwD5YCm6Pwu\_c





标准中注册的声明 \(建议但不强制使用\) ：



iss: jwt签发者

sub: jwt所面向的用户

aud: 接收jwt的一方

exp: jwt的过期时间，这个过期时间必须要大于签发时间

nbf: 定义在什么时间之前，该jwt都是不可用的.

iat: jwt的签发时间

jti: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击。



安全相关

	不应该在jwt的payload部分存放敏感信息，因为该部分是客户端可解密的部分。

	保护好secret私钥，该私钥非常重要。

	如果可以，请使用https协议

//1、自定义类型

type StandardClaims struct {

    Audience  string \`json:"aud,omitempty"\`   //预期的收件人

    ExpiresAt int64  \`json:"exp,omitempty"\`

    Id        string \`json:"jti,omitempty"\`

    IssuedAt  int64  \`json:"iat,omitempty"\`

    Issuer    string \`json:"iss,omitempty"\`

    NotBefore int64  \`json:"nbf,omitempty"\`

    Subject   string \`json:"sub,omitempty"\`

}



func \(c StandardClaims\) Valid\(\) error

Validates time based claims "exp, iat, nbf". There is no accounting for clock skew. 

As well, if any of the above claims are not in the token, it will still be considered a valid claim.



func \(c \*StandardClaims\) VerifyAudience\(cmp string, req bool\) bool

aud

func \(c \*StandardClaims\) VerifyExpiresAt\(cmp int64, req bool\) bool

exp



func \(c \*StandardClaims\) VerifyIssuedAt\(cmp int64, req bool\) bool

Compares the iat claim against cmp. If required is false, this method will return true if the value matches or is unset



func \(c \*StandardClaims\) VerifyIssuer\(cmp string, req bool\) bool

iss



func \(c \*StandardClaims\) VerifyNotBefore\(cmp int64, req bool\) bool

nbf



//2.1、创建claims

func NewWithClaims\(method SigningMethod, claims Claims\) \*Token





type Claims interface {

    Valid\(\) error

}

SigningMethodHMAC

type Token struct {

    Raw       string                 // The raw token.  Populated when you Parse a token

    Method    SigningMethod          // The signing method used or to be used

    Header    map\[string\]interface{} // The first segment of the token

    Claims    Claims                 // The second segment of the token

    Signature string                 // The third segment of the token.  Populated when you Parse a token

    Valid     bool                   // Is the token valid?  Populated when you Parse/Verify a token

}

//获取完整的签名令牌

func \(t \*Token\) SignedString\(key interface{}\) \(string, error\)

func \(t \*Token\) SigningString\(\) \(string, error\)

//生成签名字符串。 这是整个交易中最昂贵的部分。 除非你需要这个特殊的东西，直接去 SignedString 。





//HS256 说明这个令牌是通过HMAC-SHA256签名的。

//实现HMAC-SHA系列签名方法的签名方法

type SigningMethod interface {

    Verify\(signingString, signature string, key interface{}\) error // Returns nil if signature is valid

    Sign\(signingString string, key interface{}\) \(string, error\)    // Returns encoded signature or error

    Alg\(\) string                                   // returns the alg identifier for this method \(example: 'HS256'\)

}

var \(

    SigningMethodHS256  \*SigningMethodHMAC

    SigningMethodHS384  \*SigningMethodHMAC

    SigningMethodHS512  \*SigningMethodHMAC

    ErrSignatureInvalid = errors.New\("signature is invalid"\)

\)

type SigningMethodHMAC struct {

    Name string

    Hash crypto.Hash

}

func \(m \*SigningMethodHMAC\) Alg\(\) string

func \(m \*SigningMethodHMAC\) Sign\(signingString string, key interface{}\) \(string, error\)

func \(m \*SigningMethodHMAC\) Verify\(signingString, signature string, key interface{}\) error







//解析



tokenString := "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmb28iOiJiYXIiLCJleHAiOjE1MDAwLCJpc3MiOiJ0ZXN0In0.HE7fK0xOQwFEr4WDgRWj4teRPZ6i3GLwD5YCm6Pwu\_c"

type MyCustomClaims struct {

    Foo string \`json:"foo"\`

    jwt.StandardClaims

}

// sample token is expired.  override time so it parses as valid

at\(time.Unix\(0, 0\), func\(\) {

    token, err := jwt.ParseWithClaims\(tokenString, &MyCustomClaims{}, func\(token \*jwt.Token\) \(interface{}, error\) {

        return \[\]byte\("AllYourBase"\), nil

    }\)



    if claims, ok := token.Claims.\(\*MyCustomClaims\); ok && token.Valid {

        fmt.Printf\("%v %v", claims.Foo, claims.StandardClaims.ExpiresAt\)

    } else {

        fmt.Println\(err\)

    }

}\)







func \(p \*Parser\) ParseWithClaims\(tokenString string, claims Claims, keyFunc Keyfunc\) \(\*Token, error\)

type Keyfunc func\(\*Token\) \(interface{}, error\)





















