# 市场部直播APP应用接口说明

[TOC]

## 接口规范

### 协议
`HTTP/1.1`

### 请求方法

#### `GET`
一般在请求服务器资源时使用。

包含参数时,参数一律以 **Query String** 的形式传递，示例可参考**[请求手机验证码](#get_auth_code)** API。

#### `POST`
一般在操作服务器资源时使用。

包含参数时，请求头需添加`Content-Type: application/json`，参数一律以 **JSON** 格式在请求体中传递，示例可参考**[根据手机验证码登录](#login_via_auth_code)**API。

<a name="authentication"></a>
### 鉴权
用户登录后的所有操作都需要经过鉴权，鉴权方式使用 **[HTTP Basic Auth](https://zh.wikipedia.org/wiki/HTTP%E5%9F%BA%E6%9C%AC%E8%AE%A4%E8%AF%81)**。

用户在登录成功后，服务器会返回唯一的`api_key`。之后客户端的每一次请求都需要在请求头中携带经过 [Base64](https://zh.wikipedia.org/wiki/Base64) 编码的`api_key`，形式为：`Authorization: base64encode(api_key)`。

### 响应
服务器一旦接受了客户端的请求，一定会返回状态为`200 OK`的 HTTP 响应，**但请求执行结果需要根据响应内容来进行判断**。

响应内容的格式一律为 **JSON**，并一定包含两个基本的**状态码**`code`和`desc`。`code`是服务器制定的状态编码，用于指示本次返回请求所对应的结果；`desc`是对`code`的描述，用于帮助客户端的开发人员直观的理解`code`的含义而无需查表。

例如，当因为没有携带授权信息而导致请求失败时，会返回：

```
{
	"code": 4010,
	"desc": "unauthorized"
}
```

当成功时，则返回的数据中一定包含以下两项内容，以及根据不同的API规范而返回的不同数据结构：

```
{
	"code": 2000,
	"desc": "ok"
}
```
全部的状态码请参考[状态码列表](#code-list)。
全部的API接口请参考[API列表](#api-list)

###<a name='token-life-circle'></a> 令牌生命周期
令牌生命周期是指一个用户自登录之后所获得的令牌的有效时长。

**V1.0.0** 版本中设计为：一个令牌自颁发以后，除非用户**主动注销**或者**重新登录**，否则**一直有效**。

<a name='code-list'></a>
## 状态码
|状态码|返回值`code`|默认描述`desc`|说明|
|:----|:----:|:-----|:--|
|`API_OK`|2000|ok|成功|
|`API_BAD_REQUEST`|4000|bad request|请求的参数缺失或格式不符|
|`API_UNAUTHORIZED`|4010|unauthorized|未授权，一般是因为`api_key`不正确|
|`API_INVALID_AUTH_CODE`|4011|invalid auth code|错误的手机验证码|
|`API_USER_NOT_FOUND`|4041|user not found|用户未找到|

<a name='api-list'></a>
## API

### 登录

<a name='get_auth_code'></a>
#### 获取手机验证码
一个用户一次请求所获得的一个验证码，在**十分钟内有效**。

**请求**

```
GET /auth/code?mobile=<mobile>
```
- `mobile` 用户的手机号

**成功**

```
{
	"code": 2000,
	"desc": "ok"
}
```
此时服务器端以成功向用户手机发送验证码。

**失败**

```
{
	"code": 4000,
	"desc": <string desc>
}
```
- `desc`： `string`类型，根据不同场景可能表达以下含义
	- 缺少参数`mobile`
	- `mobile`格式不正确，无法获取手机验证码

<a name='login_via_auth_code'></a>
#### 登录（校验手机验证码是否正确）
**请求**

```
POST /auth/login
Content-Type: application/json

{
	"mobile": <string mobile>,
	"auth_code": <string auth_code>
}
```
- `mobile`： `string`类型，用户的手机号
- `auth_code`： `string`类型，用户的验证码

**成功**

```
{
  "code": 1000, 
  "desc": "ok", 
  "api_key": <string api_key>, 
  "mobile": <string mobile>
}
```
表示用户通过验证，此时请求中的`auth_code`过期作废。

- `api_key`： `string`类型，用户本次[生命周期](#token-life-circle)中用于[鉴权](#authentication)的秘钥
- `mobile`：`string`类型，用户登录使用的手机号码

**失败**

```
{
	"code": 4011,
	"desc": "invalid auth code"
}
```
表示用户填写的验证码错误。

#### 登出
**请求**

```
POST /auth/logout
Authorization: Basic Auth
```

**成功**

```
{
	"code": 2000,
	"desc": "ok"
}
```
登出成功。

**失败**

```
{
	"code": 4010,
	"desc": "unauthorized"
}
```
未带授权，登出失败。