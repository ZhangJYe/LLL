

# gozero微服务开发流程

## 定义API接口（Http服务）

28个HTTP接口，分为7个模块

### 基础

**无需认证（No Authentication Required）** 的接口是指：

**任何能够通过网络访问到该服务的人或系统，都可以直接调用这个 API 并获取响应，而不需要提供任何身份凭证（如 Token、用户名/密码、API Key 等）。**



在 RESTful 微服务架构中，**HTTP 方法（Method）** 就像是动词，而 **URL** 是名词。

- **URL (资源):** `/api/users` (指代“用户”这个东西)
- **Method (动作):** `GET`, `POST`, `DELETE` (指代你对“用户”想做什么)

核心HTTP方法：

| **方法**   | **英文全称** | **对应的动作**        | **是否携带数据体 (Body)** | **幂等性 (Idempotent)\*** | **典型场景**           |
| ---------- | ------------ | --------------------- | ------------------------- | ------------------------- | ---------------------- |
| **GET**    | GET          | **查询 (Read)**       | 否 (参数在 URL 中)        | ✅ 是                      | 获取用户信息、获取列表 |
| **POST**   | POST         | **创建 (Create)**     | ✅ 是 (JSON 数据)          | ❌ 否                      | 注册新用户、下单       |
| **PUT**    | PUT          | **整体更新 (Update)** | ✅ 是 (完整数据)           | ✅ 是                      | 修改用户所有信息       |
| **PATCH**  | PATCH        | **局部更新 (Modify)** | ✅ 是 (部分数据)           | ❌ 否 (通常视作非)         | 只修改密码、只修改昵称 |
| **DELETE** | DELETE       | **删除 (Delete)**     | ❌ 否                      | ✅ 是                      | 删除订单、注销账户     |

什么是“幂等性” (Idempotency)？

这是微服务中非常重要的概念。**幂等**意味着：**无论你调用这个接口 1 次还是 100 次，资源的状态都是一样的。**

- **GET 是幂等的：** 读多少次，数据都不会变。
- **DELETE 是幂等的：** 删 1 次和删 10 次，结果都是“该资源不存在”。
- **POST 不是幂等的：** 你提交 2 次“下单”请求，就会生成 2 个订单（也就是扣两次钱！）。所以在微服务中处理 POST 请求要格外小心重复提交。



URL 是地址，Method 是动作，那微服务最关键的一步来了：**怎么把“包裹”里的东西取出来？**

**URL (Uniform Resource Locator):** 是**地址**（或者说是“门牌号”）。它告诉程序**“东西在哪里”**。

**HTTP Method:** 是**动作**（或者说是“指令”）。它告诉程序**“对这个东西做什么”**。

**Request Body / Query:** 这才是真正的**“输入数据”**（原材料）。

在 Golang (Gin) 中，这个过程叫做 **“参数绑定” (Binding)**。

微服务之间通信通常使用 **JSON** 格式。Gin 框架提供了一个超级好用的功能，可以自动把 **JSON 数据** “变身” 成 **Golang 的结构体 (Struct)**。

比如，写一个登录接口：

- **URL:** `/api/login`
- **Method:** `POST`
- **输入:** 用户名和密码 (JSON)

首先，定义包裹的格式

我们要告诉 Golang，我们期待什么样的输入数据。

```
// 定义请求参数结构体
// `json:"user"` 这种叫 Tag，告诉 Go 如何匹配 JSON 里的字段
type LoginRequest struct {
    User     string `json:"user" binding:"required"` // 必填，没有就会报错
    Password string `json:"password" binding:"required"` 
}
```

编写处理逻辑

```
package main
import(
"net/http"
""github.com/gin-gonic/gin""
)
// 定义结构体（放在外面，方便复用）
type LoginRequest struct {
    User     string `json:"user" binding:"required"`
    Password string `json:"password" binding:"required"`
}
func main(){
	r:=gin.Default()
	r.POST("api/login",func(c *gin.Context)){
		var req LoginRequest
		// 核心步骤：ShouldBindJSON
        // 它会自动读取 Body，解析 JSON，填入 req 变量，还能顺便检查必填项！
        if err := c.ShouldBindJSON(&req); err != nil {
            // 如果解析失败（比如没传密码），直接返回 400 错误
            c.JSON(http.StatusBadRequest, gin.H{"error": "参数错误: " + err.Error()})
            return
        }
        // 到这里，req.User 和 req.Password 已经有值了
        // 模拟登录逻辑
        if req.User == "admin" && req.Password == "123456" {
            c.JSON(http.StatusOK, gin.H{"status": "登录成功", "token": "xyz-token-123"})
        } else {
            c.JSON(http.StatusUnauthorized, gin.H{"status": "账号或密码错误"})
        }
	})
	r.Run(":8080")
}
```



简单来说，**Tag（标签）就像是给结构体的字段贴了一张“便利贴”或者“翻译对照表”。**

这里有一个 Golang 和 JSON 之间的“语言不通”问题：

- **Golang 的规矩：** 如果你想让一个字段能被外部（比如 JSON 库）访问，这个字段的首字母**必须大写**（例如 `UserName`）。在 Go 语言里，大写代表“公开（Public）”。
- **JSON 的习惯：** 在前端或 API 传输中，大家通常习惯用**小写**或者**下划线**（例如 `username` 或 `user_name`）。

如果我不加 Tag，Go 会直接把大写的 `UserName` 转成 JSON 里的 `"UserName"`。

前端看到会抱怨：“我们要的是 `username`，你给我大写的干嘛？”

Tag 是如何工作的？（翻译官模式）

`json:"user"` 这张便利贴就是告诉 Golang 的 JSON 处理工具（`encoding/json` 包）：

> “嘿，虽然我在 Go 代码里叫 `UserName`（为了大写导出），但在转换成 JSON 时，请把我改名为 `user`。”

```
type Student struct {
    ID      int
    Name    string
    Age     int
}

// 转换结果（Key 也是大写的，前端可能不开心）：
// {"ID": 1, "Name": "Gemini", "Age": 18}


type Student struct {
    ID      int    `json:"id"`       // Go里叫ID，JSON里叫 id
    Name    string `json:"username"` // Go里叫Name，JSON里叫 username
    Age     int    `json:"age"`      // Go里叫Age，JSON里叫 age
}

// 转换结果（变成小写了，符合规范）：
// {"id": 1, "username": "Gemini", "age": 18}
```

Tag 还能做什么？

除了重命名，Tag 还有两个非常常用的“魔法指令”：

(1) 忽略字段 (`-`)

有些字段（比如**密码**），你只希望在 Go 代码里用，**绝对不能**通过 JSON 发送给前端。

```
type User struct {
    Name     string `json:"name"`
    Password string `json:"-"`  // 魔法指令：JSON 转换时直接无视我！
}
// 结果：{"name": "Gemini"} 
// Password 字段凭空消失了，非常安全。
```

(2) 空值省略 (`omitempty`)

如果是“选填项”，比如“昵称”，如果用户没填（空字符串），你希望 JSON 里干脆不要出现这个字段，节省流量。

```
type User struct {
    Name     string `json:"name"`
    Nickname string `json:"nickname,omitempty"` // 魔法指令：如果是空的，就别打包我了
}

// 如果 Nickname 有值：{"name": "G", "nickname": "Little G"}
// 如果 Nickname 是空：{"name": "G"}  <-- nickname 字段直接不传了
```

`json:"..."` 就是 **Go 结构体** 和 **JSON 数据** 之间的**映射规则**。

- **Go** 负责定义数据在**内存**里长什么样（通常大写驼峰）。
- **Tag** 负责定义数据在**网线**上传输时长什么样（通常小写蛇形）。



统一响应封装 (Standardized Response) —— **“体面的回复”**

**现状：** 我们现在的代码里 `c.JSON(200, gin.H{...})` 写得很随意。有的返回 `{"status": "ok"}`, 有的返回 `{"error": "fail"}`。

**痛点：** 前端对接会疯掉。前端希望不管成功失败，都能收到固定的格式，比如：

```
{
  "code": 0,          // 0代表成功，其他代表错误码
  "msg": "success",   // 提示信息
  "data": { ... }     // 真正的数据
}
```

- **需要学：** 如何封装一个 `Result` 结构体和 `Response` 方法，让所有接口返回格式整齐划一。

我们希望接口长什么样？

无论你的业务逻辑成功还是失败，前端收到的 JSON 永远应该是下面这种**固定格式**：

```
// 成功时
{
  "code": 0,            // 业务状态码：0 代表成功
  "msg": "success",     // 提示信息
  "data": {             // 真正的数据包裹
    "user_id": 123,
    "name": "Gemini"
  }
}

// 失败时
{
  "code": 1001,         // 业务状态码：非0 代表特定错误
  "msg": "密码错误",     // 错误原因
  "data": null          // 失败没数据
}
```

**核心三要素：**

1. **Code (int):** 给机器看的。前端通过 `if code == 0` 判断是否成功。
2. **Msg (string):** 给用户/开发人员看的。用于弹窗提示或调试。
3. **Data (any):** 实际的业务数据。

第一步：定义“万能”结构体

在 Golang 中，我们需要定义一个结构体来映射上面的 JSON。

这里你会学到一个新概念：**`interface{}` (空接口)**。

 在 Go 语言里，`interface{}` 就相当于 Java 的 `Object` 或 TypeScript 的 `any`。

它表示**“可以是任何类型的数据”**。

```
package main

// Response 是我们所有接口统一返回的结构
type Response struct {
	Code int         `json:"code"` // 业务码
	Msg  string      `json:"msg"`  // 提示信息
	Data interface{} `json:"data"` // 数据，用 interface{} 因为它可能是 User，可能是 []Order，也可能是 string
}
```

第二步：编写“包装工具” (Helper Functions)

为了避免每次写接口都要手动敲 `Code: 0, Msg: "..."`，我们把这些重复劳动封装成函数。

通常我们会创建一个 `common` 或 `utils` 包，但现在为了方便演示，我们先写在一起：

```
import (
	"net/http"
	"github.com/gin-gonic/gin"
)

// 成功时的快捷方法
func ResultSuccess(c *gin.Context, data interface{}) {
	c.JSON(http.StatusOK, Response{
		Code: 0,
		Msg:  "success",
		Data: data,
	})
}

// 失败时的快捷方法
func ResultFail(c *gin.Context, msg string) {
	c.JSON(http.StatusOK, Response{
		Code: 1001, // 这里可以定义更复杂的错误码枚举
		Msg:  msg,
		Data: nil,
	})
}
```



配置管理 (Configuration) —— **“不要硬编码”**

- **现状：** 端口 `:8080`、数据库密码都写死在代码里。
- **痛点：** 换个环境（比如上线到测试服）就得改代码重新编译。
- **你需要学：** 使用 **Viper** 库。它可以让你把配置写在 `config.yaml` 里，代码自动读取。



 数据库 ORM (GORM) —— **“跟数据库对话”**

- **现状：** 我们现在的数据是假的。
- **痛点：** 真实业务必须存库。
- **你需要学：** **GORM**。这是 Go 语言中最流行的 ORM 库，让你不用写复杂的 SQL 语句，直接操作 Go 结构体就能增删改查。



### 健康检查模块

#### 无需认证的接口

| 接口   | 方法 | 路径          | 说明         |
| :----- | :--- | :------------ | :----------- |
| Health | GET  | `/api/health` | 服务健康检查 |
| Ready  | GET  | `/api/ready`  | 服务就绪检查 |
| Live   | GET  | `/api/live`   | 服务存活检查 |

返回实例：

```
{
  "status": "ok",
  "message": "Service is healthy"
}
```

### **认证模块** (user-rpc 后端)

#### 无需认证的接口

| 接口         | 方法 | 路径                   | 请求体                         | 说明             |
| :----------- | :--- | :--------------------- | :----------------------------- | :--------------- |
| Register     | POST | `/api/auth/register`   | `{username, password, email?}` | 用户注册         |
| Login        | POST | `/api/auth/login`      | `{username, password}`         | 用户登录         |
| RefreshToken | POST | `/api/auth/refresh`    | `{refresh_token}`              | 刷新访问令牌     |
| SendSms      | POST | `/api/auth/sms/send`   | `{phone}`                      | 发送短信验证码   |
| SmsLogin     | POST | `/api/auth/sms/login`  | `{phone, code}`                | 短信登录         |
| VerifySms    | POST | `/api/auth/sms/verify` | `{phone, code}`                | 验证短信验证码   |
| GetSmsStatus | POST | `/api/auth/sms/status` | `{phone}`                      | 获取短信发送状态 |

登录和注册响应实例

```
{
  "user_id": 1,
  "username": "testuser",
  "email": "test@example.com",
  "access_token": "eyJhbGc...",
  "refresh_token": "eyJhbGc...",
  "expires_in": 900
}
```

### 汇率模块 (exchange-rpc 后端)

#### **无需认证的接口**

| 接口             | 方法 | 路径                 | 请求体               | 说明                    |
| :--------------- | :--- | :------------------- | :------------------- | :---------------------- |
| GetExchangeRates | GET  | `/api/exchangeRates` | -                    | 获取所有汇率记录        |
| GetLatestRates   | GET  | `/api/latestRates`   | -                    | 获取最新汇率（外部API） |
| Convert          | POST | `/api/convert`       | `{from, to, amount}` | 货币转换                |









## 定义RPC服务（内部服务调用）

## 公共代码管理

## 配置管理