http有哪些头部？详细介绍一下Cookie。

http和https了解多少? 说说看

#### jwt、session、cookie 的区别
> **Cookie 是一种存储机制，Session 是基于服务器的状态管理方案，JWT 是一种基于 Token 的无状态认证方案。**
##### JWT（JSON Web Token）
**定义：** JWT 是一种自包含的**无状态 Token**，服务端通过签名验证 Token 的合法性，而不是存储状态。
**特点：**
- 客户端存储 Token，不依赖服务器记录
- 通常用于**前后端分离**、**移动端**、**微服务**认证
- 自包含信息（如用户 ID、过期时间等）
**工作流程：**
1. 用户登录成功，服务端生成 JWT 并返回给客户端
2. 客户端将 JWT 存在 Cookie 或 LocalStorage 中
3. 之后请求时带上 JWT，服务端验证签名即可
**结构组成：**
```txt
Header.Payload.Signature

// 例如：
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoxMjM0NTY3ODkwfQ.PaWcN0vK
```
**优点：**
- 无状态，适合分布式系统
- 不需要服务端存储会话数据
- 可扩展性强，支持自定义字段
**缺点：**
- 不易主动失效（不像 Session 可以服务端删除）
- Token 长度大，占用带宽
- 必须加密签名+过期机制，否则容易被伪造
##### Session（服务器状态管理）
**定义：** Session 是服务器在内存中为每个用户创建的一段会话数据，通常依赖 Cookie 存储 `session_id`。
**特点：**
- 数据保存在服务器端（安全）
- 客户端只持有 session ID
- 每个用户都有独立的 session
**工作流程：**
1. 客户端登录成功，服务端创建 session 并生成 `session_id`
2. 服务端通过 `Set-Cookie` 把 `session_id` 返回给浏览器
3. 浏览器以后每次请求都携带 `session_id`，服务端识别并获取会话数据

**缺点：**
- **依赖服务端存储**，不适合分布式架构（除非用 Redis、共享 Session）
- 用户多时内存压力大
##### Cookie（客户端状态存储）
**定义：** Cookie 是由服务器写入浏览器的一段**小型文本数据**，浏览器之后每次请求会自动携带这些数据。
**特点：**
- 存储在浏览器端，大小限制为 4KB 左右
- 通常用于记录登录信息、用户偏好等
- 有过期时间，可设置为临时或持久
**示例：**
```http
Set-Cookie: session_id=abc123; Path=/; HttpOnly
```
**安全属性：**
- `HttpOnly`：防止 JS 获取（防 XSS）
- `Secure`：仅在 HTTPS 下传输（防劫持）
- `SameSite`：限制跨站请求（防 CSRF）

#### http 和 https 的异同（区别和联系）
> HTTP 是明文传输的协议，HTTPS 是加了“安全壳”的 HTTP。

##### 联系
| 项目    | 描述                                         |
| ----- | ------------------------------------------ |
| 协议基础  | HTTPS 是以 HTTP 协议为基础的，通过 SSL/TLS 协议提供安全加密传输 |
| 请求方式  | 都支持 GET、POST、PUT、DELETE 等标准 HTTP 方法        |
| 应用层协议 | 都属于应用层协议，工作在 OSI 七层模型中的最上层                 |
| 通信模型  | 都采用客户端-服务端模型（C/S），以请求响应方式进行通信              |
##### 区别
| 对比项  | HTTP             | HTTPS                     |
| ---- | ---------------- | ------------------------- |
| 传输方式 | 明文传输，数据不加密       | 使用 SSL/TLS 加密，数据传输更安全     |
| 端口号  | 默认端口是 **80**     | 默认端口是 **443**             |
| 安全性  | 无法防止窃听、中间人攻击     | 提供加密、身份验证、防篡改、防劫持         |
| 证书支持 | 不需要证书            | 需要由 CA 机构签发的 **SSL 数字证书** |
| 性能开销 | 快，资源消耗少          | 有握手与加密解密过程，资源消耗较大         |
| 使用场景 | 一般用于无敏感数据的站点，如博客 | 用于涉及登录、支付、个人数据等安全要求高的站点   |
##### https 是如何保证安全的
HTTPS = HTTP + SSL/TLS

具体过程：
1. 客户端访问服务器，服务器返回数字证书
2. 客户端验证证书是否合法（如是否被 CA 签发）
3. 双方进行 TLS 握手，协商加密算法和密钥
4. 使用对称加密传输数据，防止数据被第三方窃听或篡改

##### 为什么推荐使用 https
- 🔒 **防止信息泄露**：防止账号密码、身份证号等敏感信息被截取
- 🛡️ **提高网站可信度**：有浏览器小锁标志，显示“安全”
- 📈 **SEO 加分项**：谷歌、百度等搜索引擎倾向于优先收录 HTTPS 站点
- ⚠️ **防止劫持**：HTTP 容易被运营商插入广告或脚本，HTTPS 可有效防止


#### jwt 的结构，和 session、cookie 的区别，解决了什么问题
##### jwt 的结构：三段式 Token
JWT（JSON Web Token）结构如下：
```txt
Header.Payload.Signature
```
例子：
```txt
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
.
eyJ1c2VySWQiOjEyMywiZXhwIjoxNjg5MjAwMDAwfQ
.
Q5nwh8OSKs4sVnG5f8NnqkA9wlZ_kTxjsvHFZqG9grI

```
**1.Header（头部）**
```json
{
  "alg": "HS256",  // 签名算法
  "typ": "JWT"
}
```
编码后是 Base64 字符串。
**2.Payload（载荷）**
```json
{
  "userId": 123,
  "role": "admin",
  "exp": 1689200000 // 过期时间
}
```
用于传递自定义信息，是**公开信息**，任何人都能解码查看，但不可篡改（因为有签名保护）。
**3.Signature（签名）**
```scss
HMACSHA256(Base64UrlEncode(header) + "." + Base64UrlEncode(payload), secret)
```
用于**验证 Token 是否被篡改**。

##### 和 session、cookie 的区别
| 特性/机制  | JWT                      | Session              | Cookie                    |
| ------ | ------------------------ | -------------------- | ------------------------- |
| 存储位置   | 客户端（LocalStorage/Cookie） | 服务端                  | 客户端（浏览器）                  |
| 状态管理   | 无状态（Stateless）           | 有状态（Session 存在服务端）   | 存储有限信息，如 session_id、偏好等   |
| 安全性    | 签名保护，但易泄露                | 服务端可控，安全性较高          | 中等，需开启 HttpOnly/Secure 防护 |
| 适配分布式  | ✅ 天然适配                   | ❌ 需要 session 共享/集中存储 | ✅ 配合 JWT 可支持分布式           |
| 是否可控失效 | ❌ 不可控，需结合黑名单机制           | ✅ 服务端可主动失效           | ✅ 设置过期时间或删除               |
| 数据体积   | 大（包含用户信息）                | 小（仅 session_id）      | 小（单条约 4KB）                |
| 易用性    | 配置复杂，需处理刷新机制             | 简单（框架常内置）            | 简单，用于数据跟随请求               |

##### 解决了什么问题
| 问题             | 传统方案存在的问题                      | JWT 的解决方案                      |
| -------------- | ------------------------------ | ------------------------------ |
| **分布式认证困难**    | Session 存储在单一服务或 Redis 中，横向扩展难 | 无状态，每个服务只需验证签名即可               |
| **服务器压力大**     | 多用户登录需要大量会话维护                  | 客户端持有 Token，服务端无压力             |
| **服务间通信认证麻烦**  | 每个服务都需要共享 Session 状态           | JWT 可直接在服务间传递使用                |
| **前后端分离难以配合**  | Session 基于 Cookie，浏览器环境强依赖     | JWT 可存在 LocalStorage、Header 携带 |
| **自定义信息传递不方便** | Session 数据在服务端，不方便扩展           | JWT Payload 可存储用户 ID、角色等       |

##### 什么时候使用 JWT？
✅ **适合场景：**
- 前后端分离架构（如 Vue + Golang/Node）
- 移动端应用（如 iOS/Android App）
- 分布式或微服务系统
- 第三方系统接口（如 OAuth 授权登录）

❌ **不推荐 JWT 的场景：**
- 后台管理系统，传统 Web 项目（推荐 Session）
- 高安全性场景，需要强控制的登录状态
- 无刷新机制，登录状态不可被主动废除（风险高）

#### jwt 一定好吗？什么场景用 session 会更好呢？
##### JWT 的优势与不足
**JWT 的优点（适用于分布式系统、前后端分离）：**

| 优点              | 描述                                      |
| --------------- | --------------------------------------- |
| ✅ **无状态**       | 服务器不保存会话数据，天然支持分布式部署、微服务架构              |
| ✅ **跨服务通信方便**   | Token 可直接在微服务间传递，无需共享 Session           |
| ✅ **前后端分离天然适配** | 客户端可存在本地（如 LocalStorage），每次请求自动携带 Token |
| ✅ **可拓展性强**     | Payload 可以携带自定义字段（如用户ID、角色等）            |

**JWT 的缺点：**

| 缺点              | 描述                                                 |
| --------------- | -------------------------------------------------- |
| ❌ **不可主动失效**    | 无法服务端“踢用户下线”，只能等 Token 过期（除非维护 Token 黑名单）          |
| ❌ **泄露风险高**     | 一旦被劫持，攻击者可长期使用，且不可废除（除非配合刷新机制或黑名单）                 |
| ❌ **Token 体积大** | 每次请求都传，包含签名信息，占用更多带宽资源                             |
| ❌ **更新机制复杂**    | 通常需要配合 **access token + refresh token** 双 Token 模式 |

##### Session 的优势与使用场景
**Session 适合哪些场景？**

| 场景               | 理由                            |
| ---------------- | ----------------------------- |
| 👤 **传统 Web 网站** | 服务端管理会话，安全、逻辑简单，适用于表单登录、后台管理等 |
| 🔐 **对安全要求高的系统** | 服务端可立即使 Session 失效（如登出/封号）    |
| 🧱 **小型单体应用**    | 不涉及微服务，Session 足够简单高效         |
| 🛑 **短期会话需求**    | 不需长时间保持登录状态，Session 更轻便安全     |
**Session 的优点：**

| 优点            | 描述                                 |
| ------------- | ---------------------------------- |
| ✅ **易于控制**    | 服务端可随时清除、修改 Session 内容             |
| ✅ **数据保密性强**  | 客户端只拿到 session_id，敏感数据不暴露          |
| ✅ **适合服务端渲染** | Web 页面跳转天然配合 Cookie + Session 登录机制 |
| ✅ **简单易实现**   | 各种框架都内置支持（PHP、Java、Go、Python等）     |

##### 对比
| 维度     | JWT                    | Session             |
| ------ | ---------------------- | ------------------- |
| 架构适配   | 前后端分离、分布式系统、微服务        | 单体应用、服务端渲染 Web 应用   |
| 会话存储   | 客户端自持，服务端无感知           | 服务端维护，需要 Session 存储 |
| 主动失效控制 | ❌ 不方便主动失效              | ✅ 服务端可随时失效处理        |
| 安全性    | 取决于 Token 保管、是否签名、是否加密 | 服务端控制，泄漏风险低         |
| 维护复杂度  | 高（需要配合刷新机制、安全策略）       | 低（内存或 Redis 就够了）    |


#### TCP/UDP 的区别和特点   如何监听端口   tcp，udp适用场景
##### TCP/UDP 的区别和特点
| 特性    | TCP            | UDP              |
| ----- | -------------- | ---------------- |
| 是否连接  | ✅ 面向连接（三次握手）   | ❌ 无连接（即发即走）      |
| 可靠性   | ✅ 保证顺序、重传、完整性  | ❌ 不保证可靠，可能丢包/乱序  |
| 速度    | ❌ 相对慢（有确认+握手）  | ✅ 快，低延迟          |
| 传输方式  | 字节流（Stream）    | 数据报（Datagram）    |
| 消息边界  | ❌ 无边界（程序需自己切包） | ✅ 有边界（一次发送就是一个包） |
| 应用层控制 | 少（由协议保障）       | 多（开发者自己处理重传等）    |

##### 如何监听端口
1. **TCP 监听端口**
```go
ln, err := net.Listen("tcp", ":8080") // 监听 8080 端口
if err != nil {
    log.Fatal(err)
}
for {
    conn, _ := ln.Accept()            // 接受连接（三次握手）
    go handleConn(conn)
}
```
> TCP 是**面向连接的**，每个客户端都要 `Accept()` 成一个 `conn`


2. **UDP 监听端口**
```go
addr, _ := net.ResolveUDPAddr("udp", ":8080")
conn, _ := net.ListenUDP("udp", addr)
buf := make([]byte, 1024)
for {
    n, remoteAddr, _ := conn.ReadFromUDP(buf)
    go handleMsg(buf[:n], remoteAddr)
}
```
> UDP 是**无连接的**，服务端只接收报文，不用握手或维护连接

##### tcp，udp适用场景
| 场景                   | 推荐协议           | 原因             |
| -------------------- | -------------- | -------------- |
| Web 页面请求（HTTP/HTTPS） | **TCP**        | 需要可靠传输         |
| 文件传输、数据库连接           | **TCP**        | 保证数据完整性        |
| 视频直播、语音通话            | **UDP**        | 容忍部分丢包，要求低延迟   |
| 游戏实时对战               | **UDP**        | 快速响应比准确更重要     |
| DNS 查询               | **UDP**        | 快速、简单、报文小      |
| 邮件（SMTP/POP3）        | **TCP**        | 必须可靠传输         |
| 聊天应用                 | **TCP/UDP 结合** | 长连接可靠 + 实时传输结合 |
##### 总结
- **TCP** 适合**对数据传输可靠性要求高**的场景（如网页、下载、数据库）
- **UDP** 适合**对实时性要求高、可以容忍少量丢包**的场景（如语音、直播、游戏）
- **监听端口：TCP 要握手接连接，UDP 不握手直接接收数据**


#### websocket 和 sse 区别
##### 对比总览
| 特性    | **WebSocket**                        | **SSE（Server-Sent Events）**                     |
| ----- | ------------------------------------ | ----------------------------------------------- |
| 通信方向  | **双向通信** 🔁                          | **单向通信（服务器 → 客户端）** ⬇️                          |
| 建立方式  | HTTP 升级为 WS 协议（`Upgrade: websocket`） | 基于标准 HTTP 请求（`Content-Type: text/event-stream`） |
| 支持浏览器 | ✅ 所有主流浏览器                            | ✅ 支持度高，IE 不支持                                   |
| 自动重连  | ❌ 需手动实现                              | ✅ 内置支持断线重连                                      |
| 并发连接数 | 🔺 高（受 WebSocket 限制）                 | 🔻 高（基于 HTTP/1.1）                               |
| 消息格式  | 任意二进制或文本                             | 文本（UTF-8 文本事件）                                  |
| 兼容性   | 比较新，某些老设备需兼容处理                       | 更简单、兼容性好                                        |
| 使用难度  | 中等（需管理连接/心跳）                         | 简单（浏览器原生支持）                                     |
| 协议开销  | 低（首次握手后协议很轻）                         | 高（基于 HTTP）                                      |

##### 使用场景对比
| 场景               | 推荐方案        | 理由               |
| ---------------- | ----------- | ---------------- |
| 实时聊天、协同编辑        | ✅ WebSocket | 需要**双向通信**       |
| 实时行情推送、新闻流、后台日志流 | ✅ SSE       | 服务器 → 客户端，单向推送足够 |
| 低频消息推送（如订单状态）    | ✅ SSE       | 简单、开箱即用          |
| 游戏/音视频           | ✅ WebSocket | 需要低延迟、双向传输       |
| IoT 设备通信         | ✅ WebSocket | 较轻量、实时性强         |

##### 技术实现区别
**WebSocket 工作流程**
1. 客户端发起 HTTP 请求，带上`Upgrade:websocket`
2. 服务器响应`101 Switching Protocols`
3. 双方建立 WebSocket 通道
4. 之后可双向发送消息（text 或 binary）
```http
GET /ws HTTP/1.1
Upgrade: websocket
Connection: Upgrade
```


**SSE 工作流程**
1. 客户端使用 HTTP 连接访问服务器
2. 服务端持续发送 `text/event-stream` 格式数据
3. 浏览器自动接收 & 重连
```js
const source = new EventSource("/events");
source.onmessage = function(e) {
  console.log("消息：", e.data);
}
```
服务器返回内容：
```bash
data: Hello World
id: 1

data: Another message
id: 2
```


##### 对比图
```markdown
       🔄 WebSocket                    🔁 双向
   ┌────────────┐                 ┌────────────┐
   │   客户端    │ ⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄ │    服务端    │
   └────────────┘                 └────────────┘

       🔽 SSE                         ⬇️ 单向
   ┌────────────┐                 ┌────────────┐
   │   客户端    │  ⇐⇐⇐⇐⇐⇐⇐⇐⇐⇐⇐  │    服务端    │
   └────────────┘                 └────────────┘
```

##### 总结
- **WebSocket** 适合双向通信、实时互动（如聊天、游戏）
- **SSE** 适合单向消息推送、浏览器原生支持、部署简单
- 前者灵活强大，后者轻量便捷。根据业务需求选！

#### websocket 与 http 的对比
HTTP 是基于请求-响应的单向通信协议，而 WebSocket (ws) 提供持久的、双向的实时连接。前者适合传统网页加载和 API 调用，后者更适合聊天、游戏、金融行情等实时场景。

|特性|HTTP|WebSocket (ws)|
|---|---|---|
|**通信模式**|请求-响应，客户端主动发起|双向实时，客户端与服务器均可主动|
|**连接方式**|每次请求都需重新建立连接|建立一次握手后保持长连接|
|**延迟**|较高，因需频繁建立连接|较低，消息可即时推送|
|**数据传输**|基于文本 (JSON/XML/HTML)|支持文本和二进制流|
|**适用场景**|网页加载、REST API、文件下载|聊天、游戏、IoT、实时行情|
|**资源消耗**|每次请求消耗额外 TCP/HTTP 头部|长连接占用资源，但减少重复开销|
|**安全性**|HTTPS 加密传输|WSS (WebSocket Secure) 加密传输|
- **通信模式差异**
	- HTTP：典型的 _客户端请求 → 服务器响应_，完成后连接关闭。
	- WebSocket：通过一次 HTTP 握手升级协议，之后保持长连接，允许双方随时发送数据。
- **性能与延迟**
	- HTTP 每次请求都包含完整头部，适合低频交互。
	- WebSocket 消息头极小，适合高频、低延迟场景，如股票行情推送或多人在线游戏。
- **数据类型**
	- HTTP 主要传输文本格式。
	- WebSocket 可传输二进制数据，适合音视频流或 IoT 设备数据。
- **安全性**
	- 两者都支持加密：HTTP → HTTPS，WebSocket → WSS。
	- 在防火墙和代理环境下，HTTP 更容易通过；WebSocket 有时需额外配置。


#### websocket 可以和 http 可以共用一个端口吗？
可以，WebSocket 的设计初衷就是：**WebSocket 和 HTTP 可以共用一个端口**
##### 为什么可以共用端口？
**WebSocket 是通过 HTTP 协议“升级”建立连接的**，具体过程如下：
1. 客户端先发送一个普通的 HTTP 请求，请求头中包含：
```bash
Upgrade: websocket
Connection: Upgrade
```
2. 如果服务器支持 WebSocket，就返回：
```bash
HTTP/1.1 101 Switching Protocols
```
3. 成功后，这个连接就从 HTTP **“升级为” WebSocket 协议**，后续不再是 HTTP，而是持续的 TCP 通道传输 WebSocket 数据。

**示例握手过程**
请求（客户端）：
```http
GET /ws HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: xxxxx==
Sec-WebSocket-Version: 13
```
响应（服务端）：
```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: xxxxx==
```
> 一旦连接建立，WebSocket 数据就是 **TCP 上独立协议**，不会再走 HTTP 路由。

##### 如何共用端口？
1. **Web 框架支持 WebSocket**
```go
http.HandleFunc("/", normalHandler)
http.HandleFunc("/ws", websocketHandler)

http.ListenAndServe(":8080", nil)
```
2. **Nginx 反向代理 WebSocket 和 HTTP**
可以在同一个端口上做 Nginx 分发：
```nginx
server {
    listen 80;

    location /ws/ {
        proxy_pass http://localhost:8081;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
    }

    location / {
        proxy_pass http://localhost:8080;
    }
}
```

#### socket 和 websocket 之间的区别
> **Socket** 是一种通信接口，是操作系统提供的底层“通信抽象”**；  
   WebSocket 是建立在 Socket 基础之上，专为浏览器和服务端双向通信而设计的一种高层协议

| 对比项     | Socket               | WebSocket                    |
| ------- | -------------------- | ---------------------------- |
| 📖 定义   | 一种网络通信机制/接口（API）     | 一种基于 TCP 的通信协议               |
| 📦 属于   | 操作系统层面的编程接口          | 应用层协议（基于 HTTP/1.1 升级）        |
| 🚀 协议支持 | TCP / UDP / SCTP 等协议 | 基于 TCP                       |
| 🎯 作用   | 实现通信能力（传输字节流）        | 为浏览器和服务端提供**双向通信协议**         |
| 📡 建立过程 | 手动创建连接（如 TCP 三次握手）   | HTTP 请求后升级为 WebSocket        |
| 🛠 使用方式 | 编程语言直接调用系统 socket 接口 | 浏览器内建支持，`new WebSocket(url)` |
| 💬 消息格式 | 没有固定格式，自定义           | 有协议头、帧格式、ping/pong           |




