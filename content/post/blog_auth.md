---
title: "从个人博客搭建谈用户认证"
date: 2020-10-19T14:44:49+08:00
draft: false
toc: true
author: "彬Goh"
keywords: ["登陆","jwt","token","session"]
description: "从个人博客搭建谈用户认证"
tags: ["用户认证"]
categories: ["开发-后端"]
---
假设我们现在需要搭建一个个人博客，除了展示给访问者的页面之外，还有一个后台管理系统。管理者通过用户名密码登陆，然后在后台管理博文、评论、标签等。
<!--more-->
前台展示是所有访问者都能获取的，而后台管理只有登陆之后才能获取相关权限。众所周知，`http`是一个无状态协议，所以通过登录页面进入后台管理页面之后，服务器并不会“记住”你曾经登陆过。那我们怎么保证后台管理页面只有登陆的人才能访问，而且不需要每次访问都让用户输入用户名和密码呢？

这就是用户认证机制。一般我们使用`session`或者`token`来进行用户验证。


## session

用户第一次登陆时，需要输入用户名和密码。服务端在对用户名密码验证之后，生成`session`，并返回一个`session id`，存储在`cookie`中。之后用户再点击后台管理页面的其他接口时，需要带上这个`session id`。服务端通过查看用户的`seesion id`来判断用户的登录状态与个人信息。

使用 golang 的 gin 框架与`"github.com/go-session/session"`包来做一个小小的 demo。个人博客的接口如下：

| 路由            | 请求类型 | 功能         |
| --------------- | -------- | ------------ |
| `/`             | GET      | 前台展示页面 |
| `/display`      | GET      | 前台展示页面 |
| `/login`        | POST     | 登陆页面     |
| `/admin/blog`   | GET      | 后台管理博文 |
| `/admin/logout` | GET      | 登出         |

### 前台页面

这个页面并不需要登陆状态，任何人可以访问

```go
// 基础设置
router := gin.Default()
session.InitManager(
    session.SetCookieName("session_id"), // cookie 的名字
    session.SetSign([]byte("sign")), // 签名
)

// 前台展示界面
router.GET("/", hello)
router.GET("/display", hello)

// 监听 8080 端口
router.Run("localhost:8080")
```

访问`/`和`/display`时返回问候语：

```go
// "/display"
func hello(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{
        "message": "this is display page! hello!",
    })
}
```

用postman访问`127.0.0.1:8080/display`或者`127.0.0.1:8080`，得到响应如下：

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gjsfkly7msj31jg0ai75s.jpg)

### 登陆页面

需要获得用户名和密码，检验是否可以登陆。登陆成功之后重定向到个人博客管理后台。

```go
// 登陆
router.POST("/login", login)
```

登陆使用`post`请求。

```go
type Info struct {
    UsrName  string `json:"usr_name"`
    Password string `json:"password"`
}

// "/login"
func login(c *gin.Context) {
    // 绑定请求体，得到用户名和密码
    info := Info{}
    c.BindJSON(&info)

    // 验证用户名密码
    if info.UsrName == "admin" && info.Password == "123456" {
        // 登陆成功， 生成一个会话
        store, _ := session.Start(context.Background(), c.Writer, c.Request)
        store.Set("usrName", info.UsrName)
        store.Save()
        // 重定向到博文管理
        c.Redirect(http.StatusMovedPermanently, "/admin/blog")
    } else {
        // 登陆失败
        c.JSON(http.StatusUnauthorized, gin.H{
            "massage": "you are not allowed to login!",
        })
    }
}

// "/admin/blog"
func adminBlog(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{
        "message": "this is your personal blog page!",
    })
}
```

当登陆密码错误时：

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gjsfylm12nj31jg0o00wa.jpg)

当登陆密码正确时，直接重定向到个人博文管理

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gjsfzrhl9oj31jc0osjuw.jpg)

仅仅是上面当然是不够的，最重要的是登陆成功之后生成了一个`session id`并返回给浏览器。

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gjsg395ms5j31120463za.jpg)

用postman查看`cookie`如上所示，并且设置了默认过期时间。当下次再用postman访问相同域名时，就会带上这个`cookie`。

### 后台管理

```go
// 后台管理界面
g := router.Group("/admin", adminCheck)
{
    g.GET("/blog", adminBlog)
    g.GET("/logout", adminLogout)
}

// "/admin"
func adminCheck(c *gin.Context) {
    // 检验登陆状态
    store, _ := session.Start(context.Background(), c.Writer, c.Request)
    _, ok := store.Get("usrName")
    if ok {
        // 登陆成功，继续执行逻辑
        c.Next()
    } else {
        // 登陆失败，停止
        c.JSON(http.StatusUnauthorized, gin.H{
            "massage": "you are not allowed!",
        })
        c.Abort()
    }
}

// "/admin/logout"
func adminLogout(c *gin.Context) {
    store, _ := session.Start(context.Background(), c.Writer, c.Request)
    store.Delete("usrName")
    c.JSON(http.StatusOK, gin.H{
        "message": "logout!",
    })
}
```

对于所有前缀是 `/admin`得到接口，都认为是需要登陆的。所以加上中间件，进行登陆状态的检验。`登陆`是在`session`中存储键值对，那么`检验登陆`就是查看这个键值对是否存在。如果不存在，则说明不是登陆状态。例如我们这里存储了`userName`，那么这个值可以取出来供后续`CRUD`使用。而登出时删除这个键值对即可。

在未登陆的时候直接访问`/admin/blog`，请求被拒绝。

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gjsgc6au5oj31jk0o80w4.jpg)

在执行`/login`并成功登陆之后，请访问成功！在登陆之后请求`/logout`，退出成功，并且不能再访问`/admin/blog`。

### 源码阅读

创建和获得一个`session`的函数如下所示：

```go
session.Start(context.Background(), c.Writer, c.Request)
```

处理逻辑如下：

1. 根据请求获得 `session id`

```go
sid, err := m.sessionID(r) // 先不必在意 m 是什么，知道这个是获取 session id 即可
```

2. 如果 `session id` 不为空，则在内存中找这个`id`下存储的键值对

```go
// 如果 session id 不为空， 则检验它
if sid != "" {
   if exists, verr := m.opts.store.Check(ctx, sid); verr != nil {
      return nil, verr
   } else if exists {
      return m.opts.store.Update(ctx, sid, m.opts.expired) // 重新设置过期时间
}
```

上面的 `Check`函数实现如下：

```go
func (s *memoryStore) Check(_ context.Context, sid string) (bool, error) {
    // 加锁，并获取 sid 的内容
    s.RLock()
    item, ok := s.data[sid]
    s.RUnlock()

    // 如果能找到，并且没有过期则返回 true
    if ok && item.expiredAt.After(now()) {
        return true, nil
    }
    return false, nil
}
```




如果这个`session`是存在并且没有过期的，则重新设置过期时间。

3. 如果  `session id` 不存在或者过期了，则创建一个新的`session`，同时把`session id`写到`cookie`中。

```go
store, err := m.opts.store.Create(ctx, m.opts.sessionID(), m.opts.expired) // 创建一个 session
m.setCookie(store.SessionID(), w, r)
```

## token

`session`的问题在于它需要服务器开辟内存空间来存储键值对。如果是单机，只是占用了内存；但如果是分布式，那么数据同步就会非常麻烦。同时如果把`session`相关的内容存在数据库里，查询效率非常低。所以另一种被广泛运用的用户认证方式就是`token`。这里讲一讲一种`token`实现方式：`jwt`：`json web token`。

JWT 分成3个部分：

1. Header
2. Payload
3. Verify Signature

Header 部分说明了编码算法；Payload部分是添加的表明用户身份的信息；Verify Signature 对上述信息进行加密，得到一个字符串。然后对这三个部分分别进行编码（注意不是加密），并用`.`分割，就形成了一个`token`字符串。形如下：

```
eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJhdXRoIjoidHJ1ZSIsImV4cCI6MTYwMzA3OTAwNCwiaWQiOjF9.2bjfmbc0n-L65Axg4WsDOmtNAM2ikhLQ8cgBc75AiJd-xCwNTVS1sn7MOempKxhEdWHIaJuhe6zZ69TRLunEuw
```

在[jwt官网](https://jwt.io)用`HS512`算法对其进行解码（jwt有多种编码方式），可以得到这三个部分的数据。

```
Header:
{
  "alg": "HS512",
  "typ": "JWT"
}

PAYLOAD:
{
  "auth": "true",
  "exp": 1603079004,
  "id": 1
}
```

上面两个部分只是编码和解码，**所以可以认为是明文传输**。只有最后一个部分是加了密的。它是根据前面两个部分+你的密钥来加密。当你得到用户传过来的`jwt`时，只需要用自己的密钥对前两个部分再次加密，并对比`验证签名`部分，如果一致，就说明用户信息并未被修改，且用户是登陆过的。如果不一致，说明代表用户信息的这个部分被修改过了，那么就是不可信的，拒绝登陆。

> 这个其实和https的数字签名是一样的。在参加数学建模竞赛的时候，为了缓解大家一起提交论文的时候服务器压力过大，也是先给你的论文生成一个MD5码。参赛者先提交MD5码，之后再慢慢提交论文。如果你的论文修改了任意内容，MD5都会变化。必须保证你的MD5码和论文生成的是一致的。

这种做法有2个弊端：

1. JWT是无状态的，判断是否过期只是根据你的PAYLOAD中的字段，所以只有等它过期了之后你才能拒绝登陆，否则一旦jwt颁发出去，就一直是有效的。
2. 当token过期之后，需要重新再登陆，这使得用户体验很差。

我们可以通过以下方式来解决上述问题：

1. 存储jwt元信息，比如redis。这样如果用户登出，从redis中删除这个jwt就行了。或者是增加一个黑名单，先过滤一下。
2. 使用`refresh token`来产生新的token。给用户办法的用于登陆的`token`期限比较短，`refresh token`期限比较长。如果用户保持登陆状态，即使`token`过期，只要`refresh token`没过期，可以自动获取新的`token`，提高用户体验。

> 个人觉得用redis存储jwt相比直接session内容存在内存或db中的优点在于，存储内容比较少，而且读写更快。

### 基本登陆和验证的代码实现

#### 登陆

阅读 `github.com/dgrijalva/jwt-go`

一个简单的登陆例子：

```go
func CreateToken(id uint64) (string, error){
    // atClaims 是PAYLOAD部分
    atClaims := jwt.MapClaims{}
    atClaims["auth"] = "true"
    atClaims["id"] = id
    atClaims["exp"] = time.Now().Add(time.Minute * 15).Unix() // 默认验证的就是 "exp"字段
    // 生成一个结构体
    at := jwt.NewWithClaims(jwt.SigningMethodHS512, atClaims)
    // 签名并返回token
    token, err := at.SignedString([]byte("asjdklqwejkqjkl"))
    if err != nil {
        log.Printf("err: %s\n", err)
    }
    // 放在 response header 中
    c.Header("jwt", token)
    return token, nil
}
```

源码如下：

```go
// 生成结构体
func NewWithClaims(method SigningMethod, claims Claims) *Token {
    return &Token{
        Header: map[string]interface{}{
            "typ": "JWT", // 固定的
            "alg": method.Alg(), // 编码算法
        },
        Claims: claims,
        Method: method,
    }
}

// 签名并得到完整的token
func (t *Token) SignedString(key interface{}) (string, error) {
    var sig, sstr string
    var err error
    // 对前两个部分编码
    if sstr, err = t.SigningString(); err != nil {
        return "", err
    }
    // 签名
    if sig, err = t.Method.Sign(sstr, key); err != nil {
        return "", err
    }
    // 返回token
    return strings.Join([]string{sstr, sig}, "."), nil
}

// 对前两个部分编码
func (t *Token) SigningString() (string, error) {
    var err error
    parts := make([]string, 2)
    for i, _ := range parts {
        var jsonValue []byte
        if i == 0 {
            // Header 部分，将结构体序列化成字节
            if jsonValue, err = json.Marshal(t.Header); err != nil {
                return "", err
            }
        } else {
            // PAYLOAD 部分，将结构体序列化成字节
            if jsonValue, err = json.Marshal(t.Claims); err != nil {
                return "", err
            }
        }
        // 编码
        parts[i] = EncodeSegment(jsonValue)
    }
    // 用 . 分隔
    return strings.Join(parts, "."), nil
}
```

#### 验证

代码如下：

```go
func verifyToken(tokenString string) (*jwt.Token, error) {
    // 验证是否是颁发的token以及是否过期等信息，存在token里面
    token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
        log.Println("method", token.Method)
        if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
            return nil, fmt.Errorf("unexpected signing method: %v", token.Header["alg"])
        }
        return []byte("asjdklqwejkqjkl"), nil
    })
    if err != nil {
        log.Println("err", err)
        return nil, err
    }
    return token, nil
}
```

