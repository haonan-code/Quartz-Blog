## 1. 什么是 RESTful API
### 1.1 REST 定义
REST（Representational State Transfer，表述性状态转移）是一种 **API 设计风格**，不是协议、不是框架。
它基于 **HTTP 协议**，通过资源（Resource）的表述（Representation）进行交互。
**核心思想**：
- **一切皆资源**（Resource）
- **资源通过 URL 统一标识**
- **使用 HTTP 方法操作资源**
- **无状态通信**（Stateless）
### 1.2 RESTful API 的核心原则

| 原则               | 说明                    | 示例                         |
| ---------------- | --------------------- | -------------------------- |
| **资源（Resource）** | 用名词表示，不是动词            | `/users` 而不是 `/getUsers`   |
| **HTTP 方法**      | 用 HTTP 方法表达操作类型       | `GET /users`、`POST /users` |
| **统一接口**         | 返回 JSON，状态码统一         | `{"id":1,"name":"Tom"}`    |
| **无状态**          | 服务端不存储客户端状态，认证用 Token | JWT、OAuth2                 |
| **层级结构**         | URL 层级表达资源关系          | `/users/1/orders`          |
| **版本控制**         | 在 URL 或 Header 中体现    | `/api/v1/users`            |
| **HATEOAS**      | 超媒体驱动（高级，一般不考）        | 响应中返回可操作链接                 |

## 2. 如何设计与实现 RESTful API
通过一个 用户管理系统 例子，说明如何设计 RESTful API
### 2.1 API 设计原则
① **URL 设计**
- 用**名词**表示资源
- 用**路径层级**表示资源关系
- 避免冗余动词
```txt
✅ GET    /users           # 获取用户列表
✅ GET    /users/123       # 获取单个用户
✅ POST   /users           # 创建用户
✅ PUT    /users/123       # 更新用户
✅ DELETE /users/123       # 删除用户
```
**错误设计**：
```txt
❌ GET    /getUsers
❌ POST   /createUser
❌ GET    /userList
```
② **HTTP 方法设计**

| 方法         | 语义   | 幂等性 | 场景       |
| ---------- | ---- | --- | -------- |
| **GET**    | 查询资源 | ✔️  | 获取用户     |
| **POST**   | 创建资源 | ❌   | 新增用户     |
| **PUT**    | 更新资源 | ✔️  | 更新整条用户信息 |
| **PATCH**  | 局部更新 | ✔️  | 只更新邮箱    |
| **DELETE** | 删除资源 | ✔️  | 删除用户     |

③ **状态码设计**
**推荐统一返回格式**：
```json
{
  "code": 0,
  "message": "ok",
  "data": {...}
}
```

| 状态码 | 场景    | 示例              |
| --- | ----- | --------------- |
| 200 | 成功    | GET /users      |
| 201 | 创建成功  | POST /users     |
| 204 | 删除成功  | DELETE /users/1 |
| 400 | 参数错误  | 缺少字段            |
| 401 | 未认证   | Token 失效        |
| 403 | 无权限   | 无权访问            |
| 404 | 资源不存在 | GET /users/999  |
| 500 | 服务器错误 | DB 挂掉           |

④ **分页、排序、搜索**
- **分页**：
	`GET /users?page=1&size=20`
- **排序**：
	`GET /users?sort=created_at,desc`
- **搜索**：
	`GET /users?keyword=tom`

⑤ **错误处理**
统一异常处理，避免返回 500 堆栈信息。
例如 Go + Gin：
```go
c.JSON(http.StatusBadRequest, gin.H{
    "code": 400,
    "message": "invalid parameter",
})
```
⑥ **安全设计**
- 使用 **HTTPS**
- 使用 **JWT** / OAuth2 认证
- 限流、签名、防重放
### 2.2 实现 RESTful API（Go + Gin 示例）
```go
// 创建用户
func CreateUser(c *gin.Context) {
    var user User
    if err := c.ShouldBindJSON(&user); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"code": 400, "message": "invalid param"})
        return
    }

    if err := db.Create(&user).Error; err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"code": 500, "message": "create user failed"})
        return
    }

    c.JSON(http.StatusCreated, gin.H{
        "code":    0,
        "message": "ok",
        "data":    user,
    })
}
```

## 3. 面试高频知识点 
### 3.1 高频概念题

| 面试题                | 回答要点                             |
| ------------------ | -------------------------------- |
| 什么是 RESTful API？   | 一种 API 设计风格，基于资源、无状态、统一接口        |
| REST 和 RPC 的区别？    | REST 偏资源、HTTP 原生支持；RPC 偏方法调用、性能高 |
| RESTful API 的核心原则？ | 资源化、无状态、统一接口、使用 HTTP 方法表达语义      |
| HTTP 方法如何设计？       | GET、POST、PUT、PATCH、DELETE，掌握幂等性  |

### 3.2 高频场景题

| 场景   | 设计要点                     |
| ---- | ------------------------ |
| 用户管理 | `/users`、分页、搜索           |
| 登录认证 | JWT、Session、OAuth2       |
| 文件上传 | `POST /files`，返回 URL     |
| 分页   | `page` + `size`          |
| 批量操作 | POST /users/batch        |
| 乐观锁  | PUT /users/123?version=2 |

### 3.3 高频追问

| 提问               | 思路                              |
| ---------------- | ------------------------------- |
| 如何保证 API 向后兼容？   | 使用 `/api/v1` 版本号或 Accept Header |
| 如何设计幂等 API？      | 结合 **幂等 Key**、PUT、唯一约束          |
| 如何保证安全性？         | JWT、HTTPS、签名、防重放                |
| 如何做性能优化？         | 缓存、分页查询、限流、CDN                  |
| REST vs GraphQL？ | REST 灵活，GraphQL 更高效，按需返回字段      |
