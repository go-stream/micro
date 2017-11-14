使用填充解码特定的jwt base64url编码

func DecodeSegment\(seg string\) \(\[\]byte, error\)

使用填充编码特定的jwt base64url编码

func EncodeSegment\(seg \[\]byte\) string

解析PEM编码的椭圆曲线私钥结构

func ParseECPrivateKeyFromPEM\(key \[\]byte\) \(\*ecdsa.PrivateKey, error\)

解析PEM编码的PKCS1或PKCS8公钥

func ParseECPublicKeyFromPEM\(key \[\]byte\) \(\*ecdsa.PublicKey, error\)

解析PEM编码的PKCS1或PKCS8私钥

func ParseRSAPrivateKeyFromPEM\(key \[\]byte\) \(\*rsa.PrivateKey, error\)

解析PEM编码的PKCS1或PKCS8公钥

func ParseRSAPublicKeyFromPEM\(key \[\]byte\) \(\*rsa.PublicKey, error\)

注册“alg”名称和工厂函数以进行签名方法。 这通常在init（）方法的实现中完成

func RegisterSigningMethod\(alg string, f func\(\) SigningMethod\)



对于一个类型来说，它是一个Claim对象，它必须有一个Valid方法来确定该令牌对于任何支持的原因是否是无效的

type Claims



解析方法使用此回调函数来提供验证密钥。 该函数接收解析的，但未经验证的令牌。 这使您可以使用标记的标题（如“kid”）中的属性来标识要使用哪个键。

type Keyfunc



使用map \[string\] interface {}进行JSON解码的索赔类型如果您不提供索赔类型，那么这是默认索赔类型

type MapClaims



验证基于时间的声明“exp，iat，nbf”。 没有考虑时钟歪斜。 同样，如果上述任何一项索赔不是令牌的话，它仍然被认为是有效的索赔。

func \(m MapClaims\) Valid\(\) error

将 aud claim 与cmp进行比较。 如果需要为false，则该方法将返回true，如果值匹配或未设置

func \(m MapClaims\) VerifyAudience\(cmp string, req bool\) bool

将 exp claim 与cmp进行比较。 如果需要为false，则该方法将返回true，如果值匹配或未设置

func \(m MapClaims\) VerifyExpiresAt\(cmp int64, req bool\) bool

将 iat claim 与cmp进行比较。 如果需要为false，则该方法将返回true，如果值匹配或未设置

func \(m MapClaims\) VerifyIssuedAt\(cmp int64, req bool\) bool

将 iss claim 与cmp进行比较。 如果需要为false，则该方法将返回true，如果值匹配或未设置

func \(m MapClaims\) VerifyIssuer\(cmp string, req bool\) bool

将 nbf claim 与cmp进行比较。 如果需要为false，则该方法将返回true，如果值匹配或未设置

func \(m MapClaims\) VerifyNotBefore\(cmp int64, req bool\) bool



type Parser struct {

     ValidMethods \[\]字符串//如果填充，只有这些方法将被视为有效

     UseJSONNumber bool //在JSON解码器中使用JSON数字格式

     SkipClaimsValidation bool //在令牌解析期间跳过声明验证

}

解析，验证并返回一个标记。 keyFunc将接收解析的标记，并返回验证的密钥。 如果一切都是洁净的，err将是零

func \(p \*Parser\) Parse\(tokenString string, keyFunc Keyfunc\) \(\*Token, error\)

func \(p \*Parser\) ParseWithClaims\(tokenString string, claims Claims, keyFunc Keyfunc\) \(\*Token, error\)



实现SigningMethod以添加签名或验证令牌的新方法。

type SigningMethod interface {

    Verify\(signingString, signature string, key interface{}\) error // Returns nil if signature is valid

    Sign\(signingString string, key interface{}\) \(string, error\)    // Returns encoded signature or error

    Alg\(\) string                                                   // returns the alg identifier for this method \(example: 'HS256'\)

}

从“alg”字符串获取签名方法

func GetSigningMethod\(alg string\) \(method SigningMethod\)



实现ECDSA系列签名方法的签名方法

type SigningMethodECDSA

func \(m \*SigningMethodECDSA\) Alg\(\) string

从SigningMethod实现Sign方法对于这种签名方法，key必须是ecdsa.PrivateKey结构体

func \(m \*SigningMethodECDSA\) Sign\(signingString string, key interface{}\) \(string, error\)

从SigningMethod实现Verify方法对于此验证方法，键必须是ecdsa.PublicKey结构

func \(m \*SigningMethodECDSA\) Verify\(signingString, signature string, key interface{}\) error



实现HMAC-SHA系列签名方法的签名方法

type SigningMethodHMAC

func \(m \*SigningMethodHMAC\) Alg\(\) string

从SigningMethod为此签名方法实现Sign方法。 密钥必须是\[\]字节

func \(m \*SigningMethodHMAC\) Sign\(signingString string, key interface{}\) \(string, error\)

验证HSXXX令牌的签名。 如果签名有效，则返回nil。

func \(m \*SigningMethodHMAC\) Verify\(signingString, signature string, key interface{}\) error



实现RSA系列签名方法的签名方法

type SigningMethodRSA

func \(m \*SigningMethodRSA\) Alg\(\) string

从SigningMethod实现Sign方法对于此签名方法，必须是rsa.PrivateKey结构。

func \(m \*SigningMethodRSA\) Sign\(signingString string, key interface{}\) \(string, error\)

从SigningMethod实现Verify方法对于此签名方法，必须是rsa.PublicKey结构。

func \(m \*SigningMethodRSA\) Verify\(signingString, signature string, key interface{}\) error



实现RSAPSS系列签名方法的签名方法

type SigningMethodRSAPSS

从SigningMethod实现Sign方法对于这种签名方法，key必须是一个rsa.PrivateKey结构体

func \(m \*SigningMethodRSAPSS\) Sign\(signingString string, key interface{}\) \(string, error\)

从SigningMethod实现Verify方法对于此验证方法，键必须是rsa.PublicKey结构

func \(m \*SigningMethodRSAPSS\) Verify\(signingString, signature string, key interface{}\) error



声明部分的结构版本，参见https://tools.ietf.org/html/rfc7519\#section-4.1请参阅示例以了解如何将您的声明类型

type StandardClaims struct {

    Audience  string \`json:"aud,omitempty"\`

    ExpiresAt int64  \`json:"exp,omitempty"\`

    Id        string \`json:"jti,omitempty"\`

    IssuedAt  int64  \`json:"iat,omitempty"\`

    Issuer    string \`json:"iss,omitempty"\`

    NotBefore int64  \`json:"nbf,omitempty"\`

    Subject   string \`json:"sub,omitempty"\`

}

验证基于时间的声明“exp，iat，nbf”。 没有考虑时钟歪斜。 同样，如果上述任何一项索赔不是令牌的话，它仍然被认为是有效的索赔。

func \(c StandardClaims\) Valid\(\) error

将 aud claim 与cmp进行比较。 如果需要为false，则该方法将返回true，如果值匹配或未设置

func \(c \*StandardClaims\) VerifyAudience\(cmp string, req bool\) bool

将 exp claim 与cmp进行比较。 如果需要为false，则该方法将返回true，如果值匹配或未设置

func \(c \*StandardClaims\) VerifyExpiresAt\(cmp int64, req bool\) bool

将 iat claim 与cmp进行比较。 如果需要为false，则该方法将返回true，如果值匹配或未设置

func \(c \*StandardClaims\) VerifyIssuedAt\(cmp int64, req bool\) bool

将 iss claim 与cmp进行比较。 如果需要为false，则该方法将返回true，如果值匹配或未设置

func \(c \*StandardClaims\) VerifyIssuer\(cmp string, req bool\) bool

将 nbf claim 与cmp进行比较。 如果需要为false，则该方法将返回true，如果值匹配或未设置

func \(c \*StandardClaims\) VerifyNotBefore\(cmp int64, req bool\) bool



根据您是否创建或解析/验证令牌，将使用不同的字段。

type Token struct {

    Raw       string                 // The raw token.  Populated when you Parse a token

    Method    SigningMethod          // The signing method used or to be used

    Header    map\[string\]interface{} // The first segment of the token

    Claims    Claims                 // The second segment of the token

    Signature string                 // The third segment of the token.  Populated when you Parse a token

    Valid     bool                   // Is the token valid?  Populated when you Parse/Verify a token

}

建一个新的令牌。 采取签名的方法

func New\(method SigningMethod\) \*Token

func NewWithClaims\(method SigningMethod, claims Claims\) \*Token

		示例使用HMAC签名方法创建，签名和编码JWT令牌

		// Create a new token object, specifying signing method and the claims

		// you would like it to contain.

		token := jwt.NewWithClaims\(jwt.SigningMethodHS256, jwt.MapClaims{

				"foo": "bar",

				"nbf": time.Date\(2015, 10, 10, 12, 0, 0, 0, time.UTC\).Unix\(\),

		}\)



		// Sign and get the complete encoded token as a string using the secret

		tokenString, err := token.SignedString\(hmacSampleSecret\)



		fmt.Println\(tokenString, err\)

		

		使用自定义声明类型创建令牌的示例 StandardClaim嵌入在自定义类型中，以便于标准声明的编码，解析和验证。

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

		

		示例（非典型）使用StandardClaims类型来解析令牌。 StandardClaims类型被设计为嵌入到自定义类型中以提供标准验证功能。 你可以单独使用它，但解析后无法检索其他字段。 请参阅CustomClaimsType示例以了解预期用法。

		mySigningKey := \[\]byte\("AllYourBase"\)



		// Create the Claims

		claims := &jwt.StandardClaims{

				ExpiresAt: 15000,

				Issuer:    "test",

		}



		token := jwt.NewWithClaims\(jwt.SigningMethodHS256, claims\)

		ss, err := token.SignedString\(mySigningKey\)

		fmt.Printf\("%v %v", ss, err\)



解析，验证并返回一个标记。 keyFunc将接收解析的标记，并返回验证的密钥。 如果一切都是洁净的，err将是零		

func Parse\(tokenString string, keyFunc Keyfunc\) \(\*Token, error\)

func ParseWithClaims\(tokenString string, claims Claims, keyFunc Keyfunc\) 

\(\*Token, error\)



	使用位域检查来分析错误类型的示例

	// Token from another example.  This token is expired

	var tokenString = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmb28iOiJiYXIiLCJleHAiOjE1MDAwLCJpc3MiOiJ0ZXN0In0.HE7fK0xOQwFEr4WDgRWj4teRPZ6i3GLwD5YCm6Pwu\_c"



	token, err := jwt.Parse\(tokenString, func\(token \*jwt.Token\) \(interface{}, error\) {

			return \[\]byte\("AllYourBase"\), nil

	}\)



	if token.Valid {

			fmt.Println\("You look nice today"\)

	} else if ve, ok := err.\(\*jwt.ValidationError\); ok {

			if ve.Errors&jwt.ValidationErrorMalformed != 0 {

					fmt.Println\("That's not even a token"\)

			} else if ve.Errors&\(jwt.ValidationErrorExpired\|jwt.ValidationErrorNotValidYet\) != 0 {

					// Token is either expired or not active yet

					fmt.Println\("Timing is everything"\)

			} else {

					fmt.Println\("Couldn't handle this token:", err\)

			}

	} else {

			fmt.Println\("Couldn't handle this token:", err\)

	}

	

	使用HMAC签名方法解析和验证令牌的示例

	// sample token string taken from the New example

	tokenString := "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmb28iOiJiYXIiLCJuYmYiOjE0NDQ0Nzg0MDB9.u1riaD1rW97opCoAuRCTy4w58Br-Zk-bh7vLiRIsrpU"



	// Parse takes the token string and a function for looking up the key. The latter is especially

	// useful if you use multiple keys for your application.  The standard is to use 'kid' in the

	// head of the token to identify which key to use, but the parsed token \(head and claims\) is provided

	// to the callback, providing flexibility.

	token, err := jwt.Parse\(tokenString, func\(token \*jwt.Token\) \(interface{}, error\) {

			// Don't forget to validate the alg is what you expect:

			if \_, ok := token.Method.\(\*jwt.SigningMethodHMAC\); !ok {

					return nil, fmt.Errorf\("Unexpected signing method: %v", token.Header\["alg"\]\)

			}



			// hmacSampleSecret is a \[\]byte containing your secret, e.g. \[\]byte\("my\_secret\_key"\)

			return hmacSampleSecret, nil

	}\)



	if claims, ok := token.Claims.\(jwt.MapClaims\); ok && token.Valid {

			fmt.Println\(claims\["foo"\], claims\["nbf"\]\)

	} else {

			fmt.Println\(err\)

	}

	

	使用自定义声明类型创建令牌的示例 StandardClaim嵌入在自定义类型中，以便于标准声明的编码，解析和验证。

	

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

	

获取完整的签名令牌	

func \(t \*Token\) SignedString\(key interface{}\) \(string, error\)

生成签名字符串。 这是整个交易中最昂贵的部分。 除非你需要这个特殊的东西，直接去SignedString。



func \(t \*Token\) SigningString\(\) \(string, error\)



如果令牌无效，则从解析错误

type ValidationError

Helper用一个字符串错误信息构造一个ValidationError

func NewValidationError\(errorText string, errorFlags uint32\) \*ValidationError

验证错误是一种错误类型

func \(e ValidationError\) Error\(\) string



用于从HTTP请求中提取JWT令牌的实用程序包。:

https://godoc.org/github.com/dgrijalva/jwt-go/request







