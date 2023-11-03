# Oauth2

> OAuth 就是一种授权机制。数据的所有者告诉系统，同意授权第三方应用进入系统，获取这些数据。系统从而产生一个短期的进入令牌（token），用来代替密码，供第三方应用使用。

使用 *Oauth2* 的三个步骤：

* 配置资源服务器
* 配置认证服务器
* 配置spring security

*Oauth2* 的4种授权模式：

* 授权码模式（authorization code）
* 隐藏模式（implicit）
* 密码模式（resource owner password credentials）
* 客户端模式（client credentials）

不管哪一种授权方式，第三方应用申请令牌之前，都必须**先到系统备案**，说明自己的身份，然后会拿到两个身份识别码：客户端 ID（client ID）和客户端密钥（client secret）。这是为了防止令牌被滥用，没有备案过的第三方应用，是不会拿到令牌的。

##### 授权码模式

第三方应用先申请一个授权码，然后再用该码获取令牌。适用于那些有后端的 Web 应用，授权码通过前端传送，令牌则是储存在后端，所有与资源服务器的通信都在后端完成。

**流程**

1.  A提供一个链接，用户点击后就会跳转到B网站，授权用户数据给A网站使用。

```
# 链接示意
https://b.com/oauth/authorize?
  response_type=code&
  client_id=CLIENT_ID&
  redirect_uri=CALLBACK_URL&
  scope=read
  
  response_type 参数表示要求返回授权码（code）
  client_id 参数让B知道是谁在请求
  redirect_uri 参数是B接受或拒绝请求后的跳转网址
  scope 参数表示要求的授权范围（这里是只读）
```

2.  用户跳转后，B网站会要求用户登录，然后询问是否同意给予A网站授权。用户表示同意，这时B网站就会跳回redirect\_uri参数指定的网址。跳转时，会传回一个授权码
```
# A的回调地址
https://a.com/callback?code=AUTHORIZATION_CODE
```
3.  A网站拿到授权码以后，就可以在后端，向B网站请求令牌。

```
    https://b.com/oauth/token?
     client_id=CLIENT_ID&
     client_secret=CLIENT_SECRET&
     grant_type=authorization_code&
     code=AUTHORIZATION_CODE&
     redirect_uri=CALLBACK_URL
     
     client_id参数和client_secret参数用来让B确认A的身份（client_secret参数是保密的，因此只能在后端发请求）
     grant_type参数的值是AUTHORIZATION_CODE表示采用的授权方式是授权码
     code参数是上一步拿到的授权码
     redirect_uri参数是令牌颁发后的回调网址
```
4.  B网站收到请求以后，就会颁发令牌。具体做法是向redirect\_uri指定的网址，发送一段 JSON 数据。
```
    {    
      "access_token":"ACCESS_TOKEN",  # 授权令牌
      "token_type":"bearer",
      "expires_in":2592000, # 有效期
      "refresh_token":"REFRESH_TOKEN",
      "scope":"read",
      "uid":100101,
      "info":{...}
    }
```

##### 隐藏模式

有些 Web 应用是纯前端应用，必须将令牌储存在前端。**RFC 6749 就规定了第二种方式，允许直接向前端颁发令牌。这种方式没有授权码这个中间步骤，所以称为（授权码）"隐藏式"（implicit）**。

**流程**

1.  A网站提供一个链接，要求用户跳转到B网站，授权用户数据给A网站使用。
```
https://b.com/oauth/authorize?
  response_type=token&
  client_id=CLIENT_ID&
  redirect_uri=CALLBACK_URL&
  scope=read
  
  response_type参数为token，表示要求直接返回令牌
```
2.  用户跳转到B网站，登录后同意给予A网站授权。这时，B网站就会跳回redirect\_uri参数指定的跳转网址，并且把令牌作为 URL 参数，传给A网站。

```
https://a.com/callback#token=ACCESS_TOKEN
```
token参数就是令牌，A 网站因此直接在前端拿到令牌

##### 密码模式

对于高度信任某个应用，RFC 6749也允许用户把用户名和密码，直接告诉该应用。该应用就使用提供的密码，申请令牌，这种方式称为"密码式"（password）

**流程**

1. A网站要求用户提供B网站的用户名和密码。拿到以后，A就直接向B请求令牌
```
https://oauth.b.com/token?
  grant_type=password&
  username=USERNAME&
  password=PASSWORD&
  client_id=CLIENT_ID
  
  grant_type参数是授权方式，这里的password表示"密码式"
  username和password是B的用户名和密码
```
2. B网站验证身份通过后，直接给出令牌。注意，这时不需要跳转，而是把令牌放在 JSON 数据里面，作为 HTTP 回应，A因此拿到令牌。
3. TokenEndpoint ：  /oauth/token
```
# 注册时使用真实IP
eureka:
  instance:
    prefer-ip-address: true # 注册时使用用ip而不是主机名称

# 微服务调用，获取到的是微服务的是微服务的IP而不是真实IP
```
