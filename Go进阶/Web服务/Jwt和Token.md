# 用户登录用jwt 还是token？

结论： 最直观的：**token** 需要查库验证 **token** 是否有效，而 **JWT** 不用查库或者少查库，直接在服务端进行校验，并且不用查库。 因为用户的信息及加密信息在第二部分 payload 和第三部分签证中已经生成，只要在服务端进行校验就行，并且校验也是**JWT**自己实现的。

# JWT和token的区别及优缺点

**一个单点登录的实现我们需要知道什么？**

我们首先应该知道的是什么是[单点登录](https://so.csdn.net/so/search?q=单点登录&spm=1001.2101.3001.7020)，单点登录是如何实现的，使用了什么技术，技术的选型、优缺点，应用和实现的步骤过程中的难点有哪些怎么解决的等。

技术的话：token与[JWT](https://so.csdn.net/so/search?q=JWT&spm=1001.2101.3001.7020)的区别、JWT和redis的区别应用、是否使用到了[消息中间件](https://so.csdn.net/so/search?q=消息中间件&spm=1001.2101.3001.7020)有的话kafka和其他MQ的比较、实现过程中的相关协议等。从其中总结对这些问题点进行总结。

# 一、结论：

最直观的：token需要查库验证token 是否有效，而JWT不用查库或者少查库，直接在服务端进行校验，并且不用查库。因为用户的信息及加密信息在第二部分payload和第三部分签证中已经生成，只要在服务端进行校验就行，并且校验也是JWT自己实现的。

# 二、TOKEN

**概念：**令牌， 是访问资源的凭证。是string类型的数据

**Token的认证流程：**

1. 用户输入用户名和密码，发送给服务器。
2. 服务器验证用户名和密码，正确的话就返回一个签名过的token（token 可以认为就是个长长的字符串），浏览器客户端拿到这个token。
3. 后续每次请求中，浏览器会把token作为http header发送给服务器，服务器验证签名是否有效，如果有效那么认证就成功，可以返回客户端需要的数据。

**特点：**这种方式的特点就是客户端的token中自己保留有大量信息，服务器没有存储这些信息。



# 三、JWT

**概念：**JWT是json web token缩写。它将用户信息加密到token里，服务器不保存任何用户信息。服务器通过使用保存的密钥验证token的正确性，只要正确即通过验证。

![img](./assets/xrvp4pjty8.png)

**组成：**JWT包含三个部分：Header头部，Payload负载和Signature签名。由三部分生成token，三部分之间用“.”号做分割。JWT通过`.`连接：

1. `Header（头部）`：`Hedaer` 部分用于描述该 `JWT` 的基本信息，比如其类型（通常是 `JWT`）以及所使用的签名算法（如 `HMAC SHA256` 或 `RSA`）。
2. `Payload（负载）`： `Payload` 部分包含所传递的声明。声明是关于实体（通常是用户）和其他数据的语句。声明可以分为三种类型：**注册声明**、**公共声明** 和 **私有声明**。

   - 注册声明：这些声明是预定义的，非必须使用的但被推荐使用。官方标准定义的注册声明有 7 个：

     | Claim（声明）        | 含义                                                |
     | -------------------- | --------------------------------------------------- |
     | iss(Issuer)          | 发行者，标识 JWT 的发行者。                         |
     | sub(Subject)         | 主题，标识 JWT 的主题，通常指用户的唯一标识         |
     | aud(Audience)        | 观众，标识 JWT的接收者                              |
     | exp(Expiration Time) | 过期时间。标识 JWT 的过期时间，这个时间必须是将来的 |
     | nbf(Not Before)      | 不可用时间。在此时间之前，JWT 不应被接受处理        |
     | iat(Issued At)       | 发行时间，标识 JWT 的发行时间                       |
     | jti(JWT ID)          | JWT 的唯一标识符，用于防止 JWT 被重放（即重复使用） |

   - **公共声明**：可以由使用 `JWT` 的人自定义，但为了避免冲突，任何新定义的声明都应已在 [IANA JSON Web Token Registry](https://www.iana.org/assignments/jwt/jwt.xhtml) 中注册或者是一个 **公共名称**，其中包含了碰撞防抗性名称（`Collision-Resistant Name`）。

   - **私有声明**：发行和使用 `JWT` 的双方共同商定的声明，区别于 **注册声明** 和 **公共声明**。

3. `Signature（签名）`：为了防止数据篡改，将头部和负载的信息进行一定算法处理，加上一个密钥，最后生成签名。如果使用的是 `HMAC SHA256` 算法，那么签名就是将编码后的头部、编码后的负载拼接起来，通过密钥进行`HMAC SHA256` 运算后的结果。

**JWT的工作流程大致如下**：

- **认证阶段**：用户向服务器提供凭证（如用户名和密码）。服务器验证凭证无误后，生成一个JWT，其中包含用户标识符和其他声明，并使用秘钥对其进行签名。
- **使用阶段**：客户端收到JWT后，可以在后续的每个请求中将其放在HTTP请求头中发送给服务器，以此证明自己的身份。
- **验证阶段**：服务器收到JWT后，会使用相同的秘钥验证JWT的签名，确保其未被篡改，并检查过期时间等其他声明，从而决定是否允许执行请求。

JWT的优势在于它的无状态性，服务器不需要存储会话信息，这减轻了服务器的压力，同时也方便了跨域认证。但需要注意的是，JWT的安全性依赖于秘钥的安全保管以及对JWT过期时间等的合理设置。

> gin框架中有Jwt使用例子
>

## Jwt实战

通过以下命令在 `Go` 程序里安装 `Go JWT` 依赖：

```
go get -u github.com/golang-jwt/jwt/v5
```

## 创建 Token（JWT） 对象

生成 `JWT` 字符串首先需要创建 `Token` 对象（代表着一个 `JWT`）。因此我们需要先了解如何创建 `Token` 对象。

`jwt` 库主要通过两个函数来创建 `Token` 对象：`NewWithClaims` 和 `New`。

### NewWithClaims 函数

`jwt.NewWithClaims` 函数用于创建一个 `Token` 对象，该函数允许指定一个签名方法和一组声明`claims`）以及可变参数 `TokenOption`。下面是该函数的签名：

```
NewWithClaims(method SigningMethod, claims Claims, opts ...TokenOption) *Token
```

- `method`：这是一个 `SigningMethod` 接口参数，用于指定 `JWT` 的签名算法。常用的签名算法有 `SigningMethodHS256`、`SigningMethodRS256`等。这些算法分别代表不同的签名技术，如 `HMAC`、`RSA`。
- `claims`：这是一个 `Claims` 接口参数，它表示 `JWT` 的声明。在 `jwt` 库中，预定义了一些结构体来实现这个接口，例如 `RegisteredClaims` 和 `MapClaims` 等，通过指定 `Claims` 的实现作为参数，我们可以为`JWT` 添加声明信息，例如发行人（`iss`）、主题（`sub`）等。
- `opts`：这是一个可变参数，允许传递零个或多个 `TokenOption` 类型参数。`TokenOption` 是一个函数，它接收一个 `*Token`，这样就可以在创建 `Token` 的时候对其进行进一步的配置。

**使用示例**

```
// https://github.com/chenmingyong0423/blog/blob/master/tutorial-code/go/jwt/create-token/new_with_claims.go
package main

import (
	"fmt"
	"github.com/golang-jwt/jwt/v5"
)

func main() {
	mapClaims := jwt.MapClaims{
		"iss": "程序员陈明勇",
		"sub": "chenmingyong.cn",
		"aud": "Programmer",
	}
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, mapClaims)
	fmt.Println(token != nil) // true
}
```

这段代码首先构建了包含发行者（`iss`）、主题（`sub`）和观众（`aud`）信息的 `MapClaims` 类型声明。

然后，通过调用 `jwt.NewWithClaims` 函数，并将 `jwt.SigningMethodHS256` 作为签名方法和之前构建的 `mapClaims` 作为参数传递，来创建了一个新的 `Token` 实例。

### New 函数

`jwt.New` 函数用于创建一个 `Token` 对象，该函数允许指定一个签名方法和可变参数 `TokenOption`。下面是该函数的源码：

```
func New(method SigningMethod, opts ...TokenOption) *Token {
    return NewWithClaims(method, MapClaims{}, opts...)
}
```

通过源码我们可以发现，该函数内部的实现通过调用 `NewWithClaims` 函数，并默认传入一个空的 `MapClaims` 对象，从而生成一个 `Token` 对象。

**使用示例**

```
// https://github.com/chenmingyong0423/blog/blob/master/tutorial-code/go/jwt/create-token/new.go

package main

import (
	"fmt"
	"github.com/golang-jwt/jwt/v5"
)

func main() {
	token := jwt.New(jwt.SigningMethodHS256)
	fmt.Println(token != nil) // true
}
```

## 生成 JWT 字符串

通过使用 `jwt.Token` 对象的 `SignedString` 方法，我们能够对 `JWT` 对象进行序列化和签名处理，以生成最终的 `token` 字符串。该方法的签名如下：

```
func (t *Token) SignedString(key interface{}) (string, error)
```

- `key`：该参数是用于签名 `token` 的密钥。密钥的类型取决于使用的签名算法。例如，如果使用 `HMAC` 算法（如 `HS256`、`HS384` 等），`key` 应该是一个对称密钥（通常是 `[]byte` 类型的密钥）。如果使用 `RSA` 或 `ECDSA` 签名算法（如 `RS256`、`ES256`），`key` 应该是一个私钥 `*rsa.PrivateKey` 或 `*ecdsa.PrivateKey`。
- 方法返回两个值：一个是成功签名后的 `JWT` 字符串，另一个是在签名过程中遇到的任何错误。

**使用示例**

```
// https://github.com/chenmingyong0423/blog/blob/master/tutorial-code/go/jwt/generage-token/generate_token.go

package main

import (
	"crypto/rand"
	"fmt"
	"github.com/golang-jwt/jwt/v5"
)

func GenerateJwt(key any, method jwt.SigningMethod, claims jwt.Claims) (string, error) {
	token := jwt.NewWithClaims(method, claims)
	return token.SignedString(key)
}

func main() {
	jwtKey := make([]byte, 32) // 生成32字节（256位）的密钥
	if _, err := rand.Read(jwtKey); err != nil {
		panic(err)
	}
	jwtStr, err := GenerateJwt(jwtKey, jwt.SigningMethodHS256, jwt.MapClaims{
		"iss": "程序员陈明勇",
		"sub": "chenmingyong.cn",
		"aud": "Programmer",
	})
	if err != nil {
		panic(err)
	}
	fmt.Println(jwtStr)
}
```

这段代码首先声明并初始化一个长度为 **32** 字节的 `byte` 切片，然后使用 `crypto/rand` 库的 `Read` 函数填充切片（即密钥），确保生成的密钥具有高强度的随机性和不可预测性。

然后，调用 `GenerateJwt` 函数，传入 `jwtKey`、`jwt.SigningMethodHS256` 签名方法和包含特定声明的 `MapClaims` 对象，以创建 `JWT` 字符串。

在 `GenerateJwt` 函数内部，它利用 `token.SignedString` 方法和提供的 `key` 生成并返回签名的 `JWT` 字符串。

## 解析 JWT 字符串

`jwt` 库主要通过两个函数来解析 `jwt` 字符串：`Parse` 和 `ParseWithClaims`。

### Parse 函数

`Parse` 函数用于解析 `JWT` 字符串，函数签名如下：

```
func Parse(tokenString string, keyFunc Keyfunc, options ...ParserOption) (*Token, error)
```

- `tokenString`：要解析的 `JWT` 字符串。
- `keyFunc`：这是一个回调函数，返回用于验证 `JWT` 签名的密钥。该函数签名为 `func(*Token) (interface{}, error)`。这种设计，有利于我们根据 `token` 对象的信息返回正确的密钥。例如我们可能有一个 `keyMap` 对象，类型为 `map`，该对象用于保存多个 `key` 的映射，通过 `Token` 对象的信息，拿到某个标识，就能通过 `keyMap` 获取到正确的密钥。
- `options`：这是一个可变参数。允许传递零个或多个 `ParserOption` 类型参数。这些选项可以用来定制解析器的行为，如设置 `exp` 声明为必需的参数，否则解析失败。

**使用示例**

```go
// https://github.com/chenmingyong0423/blog/blob/master/tutorial-code/go/jwt/parse-token/parse.go

package main

import (
	"crypto/rand"
	"errors"
	"fmt"
	"github.com/golang-jwt/jwt/v5"
	"time"
)

func ParseJwt(key any, jwtStr string, options ...jwt.ParserOption) (jwt.Claims, error) {
	token, err := jwt.Parse(jwtStr, func(token *jwt.Token) (interface{}, error) {
		return key, nil
	}, options...)
	if err != nil {
		return nil, err
	}
	// 校验 Claims 对象是否有效，基于 exp（过期时间），nbf（不早于），iat（签发时间）等进行判断（如果有这些声明的话）。
	if !token.Valid {
		return nil, errors.New("invalid token")
	}
	return token.Claims, nil
}

func main() {
	jwtKey := make([]byte, 32) // 生成32字节（256位）的密钥
	if _, err := rand.Read(jwtKey); err != nil {
		panic(err)
	}

	token := jwt.NewWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
		"iss": "程序员陈明勇",
		"sub": "chenmingyong.cn",
		"aud": "Programmer",
		"exp": time.Now().Add(time.Second * 10).UnixMilli(),
	})
	jwtStr, err := token.SignedString(jwtKey)
	if err != nil {
		panic(err)
	}

	// 解析 jwt
	claims, err := ParseJwt(jwtKey, jwtStr, jwt.WithExpirationRequired())
	if err != nil {
		panic(err)
	}
	fmt.Println(claims)
}
```

这段代码的重点是自定义的 `ParseJwt` 函数，它负责解析 `JWT` 字符串，并根据验证结果返回 `Claims` 数据和一个可能的存在的错误。`ParseJwt` 函数内部利用 `jwt.Parse` 解析 `JWT` 字符串。解析后，函数检查得到的 `token` 对象的 `Valid` 属性以确认 `Claims` 是否有效。有效性检查包括但不限于验证签名、检查 `token` 是否过期。如果 `token` 通过所有验证，函数返回 `Claims` 数据；如果验证失败（如签名不匹配或 `token` 已过期），则返回错误。

### ParseWithClaims 函数

`ParseWithClaims` 函数类似 `Parse`，函数签名如下：

```
func ParseWithClaims(tokenString string, claims Claims, keyFunc Keyfunc, options ...ParserOption) (*Token, error)
```

- `tokenString`：要解析的 `JWT` 字符串。
- `claims`：这是一个 `Claims` 接口参数，用于接收解析 `JWT` 后的 `claims` 数据。
- `keyFunc`：与 `Parse` 函数中的相同，用于提供验证签名所需的密钥。
- `options`：与 `Parse` 函数中的相同，用来定制解析器的行为.

**使用示例**

```go
// https://github.com/chenmingyong0423/blog/blob/master/tutorial-code/go/jwt/parse-token/parse_with_claims.go

package main

import (
    "crypto/rand"
    "errors"
    "fmt"
    "github.com/golang-jwt/jwt/v5"
)

func ParseJwtWithClaims(key any, jwtStr string, options ...jwt.ParserOption) (jwt.Claims, error) {
    mc := jwt.MapClaims{}
    token, err := jwt.ParseWithClaims(jwtStr, mc, func(token *jwt.Token) (interface{}, error) {
       return key, nil
    }, options...)
    if err != nil {
       return nil, err
    }
    // 校验 Claims 对象是否有效，基于 exp（过期时间），nbf（不早于），iat（签发时间）等进行判断（如果有这些声明的话）。
    if !token.Valid {
       return nil, errors.New("invalid token")
    }
    return token.Claims, nil
}

func main() {
    jwtKey := make([]byte, 32) // 生成32字节（256位）的密钥
    if _, err := rand.Read(jwtKey); err != nil {
       panic(err)
    }

    token := jwt.NewWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
       "iss": "程序员陈明勇",
       "sub": "chenmingyong.cn",
       "aud": "Programmer",
    })
    jwtStr, err := token.SignedString(jwtKey)
    if err != nil {
       panic(err)
    }

    // 解析 jwt
    claims, err := ParseJwtWithClaims(jwtKey, jwtStr)
    if err != nil {
       panic(err)
    }
    fmt.Println(claims) // map[aud:Programmer iss:程序员陈明勇 sub:chenmingyong.cn]
}
```

这段代码中的 `ParseJwtWithClaims` 函数与之前示例中的 `ParseJwt` 函数功能类似，都是负责解析 `JWT` 字符串，并根据验证结果返回 `Claims` 数据和一个可能的存在的错误。不同之处在于，`ParseJwtWithClaims` 函数内部使用了 `jwt.ParseWithClaims` 函数来解析 `JWT` 字符串，这额外要求我们提供一个 `Claims` 实例来接收解析后的 `claims` 数据。在此示例中，通过 `jwt.MapClaims` 提供了这一实例。

