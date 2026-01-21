## 为什么 FastAPI 的性能更快
> FastAPI 的性能之所以更快，主要得益于 **现代化设计 + 原生异步支持 + 高效底层库**。
### FastAPI 性能更快的核心原因
| 原因               | 说明                              |
| ---------------- | ------------------------------- |
| 异步 IO 支持         | 基于 `asyncio`，支持非阻塞 IO 和高并发      |
| 高性能框架 Starlette  | FastAPI 构建于 Starlette 框架之上，性能极高 |
| 高效数据校验库 Pydantic | 基于 Cython 优化，数据解析和验证快且强大        |
| 类型注解驱动           | 编译时就能进行**参数解析、校验和文档生成**，避免运行时开销 |
| 自动生成文档（无手写开销）    | 自动生成 Swagger 文档，无需额外模板渲染逻辑      |
| 更少上下文开销          | FastAPI 使用 async 不依赖线程，无线程切换成本  |
### 底层核心：Starlette + Pydantic
**1. Starlette：超轻量的异步 Web 框架**
- 是 FastAPI 的底层 HTTP 路由 & 异步服务处理框架
- 基于 **ASGI（异步服务网关接口）**
- 支持 `async/await`，性能远高于传统 WSGI 框架（如 Flask、Django）

**2. Pydantic：类型驱动的数据解析 + 校验**
- 使用 Python 类型注解（`str`, `int`, `List[User]`）自动生成请求校验逻辑
- 底层使用 Cython 提高运行效率，比 JSON Schema 验证快得多
- 不需要自己写 `request.form.get()` 等解析代码

## 为什么 FastAPI 现在用得多
##### 1. 时代背景推动：异步高性能是刚需
现代 Web 应用越来越要求：
- **高并发、高吞吐**
- **异步 IO 支持**（特别是爬虫、API 网关、微服务）
- **自动化文档、类型安全**
FastAPI 完全契合这一趋势：
- ✅ 异步支持 `async/await`
- ✅ 高性能（接近 Node.js）
- ✅ 自动生成 API 文档
- ✅ 参数自动验证（类型注解）
##### 2. FastAPI 的突出优势
| 特点           | 说明                                      |
| ------------ | --------------------------------------- |
| **高性能**      | 基于 Starlette + Pydantic，异步非阻塞，适合高并发     |
| **自动文档**     | 自动生成 Swagger UI + ReDoc，不用额外写文档         |
| **类型注解驱动开发** | 请求参数、响应体、校验规则，全由类型注解实现，IDE 提示友好         |
| **代码结构清晰**   | 路由、模型、参数解耦清晰，可读性强                       |
| **兼容依赖注入**   | 支持 Dependency Injection，方便写扩展组件（如鉴权、限流） |
| **测试友好**     | 自带 TestClient，轻松做单元测试                   |
| **现代架构兼容好**  | 适合微服务、GraphQL、WebSocket、gRPC 等现代服务结构    |
##### 3. 开发体验好 + 上手快（相比 Flask/Django）
- 不再手动解析 request 参数、校验类型
- IDE 提示更强（因为基于类型注解）
- 接口变更自动反映到 API 文档（自动化程度高）
- 使用 `@app.get("/path")` 的路由方式类似 Flask，几乎无学习成本
##### 4. 应用广泛、社区活跃
- 被多个大厂、开源项目广泛采用：
    - Hugging Face 🤗（Transformers API）
    - Microsoft（内部服务、AI API）
    - Netflix、Uber、Zulip 等使用场景
- 文档非常详细（多语言支持、案例丰富）
- 插件生态逐渐成熟（如 SQLModel、FastAPI-Users）
##### 5. 适用场景广泛
- RESTful API 服务
- 高并发异步接口
- 后端微服务
- WebSocket 通信
- AI/ML 模型部署
##### 6. 总结
FastAPI 是一个专为**现代 Web 应用**设计的**高性能**、**异步**、**类型安全**的 Python Web 框架，强调**开发效率**、**可读性**和**自动化**。


