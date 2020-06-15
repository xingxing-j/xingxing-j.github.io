---
title: 笔记-JWT
date: 2020-06-16 00:55:04
categories: 后端学习
tags:
 - [JWT]
 - [Token]
 - [前后端分离]
---

本文摘自辉哥笔记，JWT(Json Web Token)的简单了解及其工具类的编写

<!-- more -->

# JWT

## 传统认证流程

1、用户向服务器发送用户名和密码。
2、服务器验证通过后，在当前对话（session）里面保存相关数据，比如用户角色、登录时间等等。
3、服务器向用户返回一个 **session_id**，写入用户的 **Cookie**。
4、用户随后的每一次请求，都会通过 Cookie，将 session_id 传回服务器。
5、服务器收到 session_id，找到前期保存的数据，由此得知用户的身份。

### 问题说明

1 、当**前后端分离式开发**时 不管是html 还是android 还是 IOS 默认都**不携带Cookie** ，如果需要携带cookie需要
程序员之间交流
2、就算前端/android/IOS携带Cookie 但是也只能解决传统式项目 如果是**集群分布式** 项目发布在不同的机器
上**需要做session共享** 很麻烦
3、 如果实现单点登录 比如 从A和B网站是同一个公司的，2个网站中间有关联链接，当我从A点击到B的时候 到B
网站时默认是登录状态，B如何做到登录状态呢？ 此时如果想使用session的话 登录A时需要把session持久化
再跳转B网站时 需要查询数据库 看看有没有这个session 这样代码量很大

### 问题解决

不使用Session,此时服务器就**无状态**了，此时想做到认证，有一个思想，把一串数据让服务器能识别，保存在客户
端，客户端每次请求时 携带这一串服务器能识别的数据 服务器能识别 认证成功 不识别或者没有携带 认证失败
这个识别的数据 还不能轻易被破解 并且还不能一成不变 此时就使用JWT了

## JWT说明

```properties
eyJhbGci0iJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIi0iIxMjMONTY30DkwIiwibmFtZSI6IkpvaG4 
gRG9lIiwiaXNTb2NpYWwi00nRydWV9.
4pcPyMD0901PSyXnrXCjTwXyr4BsezdI1AVTmud2fU4
```

JWT的本质是个**Json对象**，通过Base64URL加密成一个看不懂的字符串。这个字符串有个2个点，两个点把字符串分割成了3块，三块分别表示: **Header**(头部), **Payload(**载荷)， **Signature**(签名)

### 头部

```json
//Header 部分是一个 JSON 对象，描述 JWT 的元数据，
{
"alg": "HS256", //属性表示签名的算法（algorithm），默认是 HMAC SHA256（写成 HS256）；
"type": "JWT" //JWT 令牌统一写为JWT。
}
```

### 载荷

```json
//Payload 部分也是一个 JSON 对象，用来存放实际需要传递的数据。JWT 规定了7个官方字段，供选用。
//iss (issuer)：签发人
//exp (expiration time)：过期时间
//sub (subject)：主题
//aud (audience)：受众
//nbf (Not Before)：生效时间
//iat (Issued At)：签发时间
//jti (JWT ID)：编号
//可以自定义属性
```

### 签名

```json
//Signature 部分是对前两部分的签名，防止数据篡改。
// 首先，需要指定一个密钥（secret）。这个密钥只有服务器才知道，不能泄露给用户
```

### JWT工具类

```java
@Component
public class TokenUtils {
	//密钥
	private static final String TOKEN_SECRET = "7bca149eb5c7fa9a400d9f628bd53a6b";
	//30分钟超时
	private static final long TIME_OUT = 30 * 60 * 1000;

	//加密方法
	public String sign() {
		try {
			Date expiration_time = new Date(System.currentTimeMillis() + TIME_OUT);
			Algorithm algorithm = Algorithm.HMAC256(TOKEN_SECRET);
			Map<String, Object> headerMap = new HashMap<>(2);
			headerMap.put("type", "JWT");
			headerMap.put("alg", "HS256");
			return JWT.create()
					.withHeader(headerMap)//添加头部
					.withIssuer("huige")//设置签发人
					.withSubject("就是不让你访问") //设置主题
					//.withClaim("userId", uid). //可设置自定义属性
					.withExpiresAt(expiration_time) //设置过期时间
					.sign(algorithm); //jwt签名
		} catch (Exception e) {
			return null;
		}
	}

	//解密
	public DecodedJWT verify(String token) {
		try {
			JWTVerifier verifier = JWT.require(Algorithm.HMAC256(TOKEN_SECRET)).build();
			DecodedJWT jwt = verifier.verify(token);
			return jwt;
		} catch (Exception e) {
			//解码异常
			return null;
		}
	}
}
```

#### 其他说明

使用时要引相关依赖

```xml
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java‐jwt</artifactId>
    <version>3.10.3</version>
</dependency>
```

前端要在请求头中添加相关Token，以发送axios异步请求为例

```javascript
axios.defaults.headers.Authorization = "Bearer " + localStorage.getItem("token");
```



