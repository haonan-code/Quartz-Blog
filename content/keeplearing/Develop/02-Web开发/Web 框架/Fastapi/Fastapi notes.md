#### 如何定义 url router 
方式一：使用装饰器的方式定义 url 路由
```python
@app.get("/health")
async def health_check() -> dict:
    """
    Health check endpoint
    """
    return {"status": "healthy"}
```
方式二：先声明路由组，然后路由组对象通过 `add_api_route`方法定义 url router
```python
router = APIRouter()

router.add_api_route(
    '/author',
    user_api.get_author_info,
    methods=['get'],
    summary='获取作者信息'
)
# 主应用中这侧路由，并为 router 路由组添加 `/api/v1` 前缀
app.include_router(router, prefix="/api/v1")
```
`add_api_route()`说明：
- **path**: 路由的路径（例如 "/items/"）。
- **endpoint**: 路由对应的处理函数（例如 get_items）。
- **methods**: HTTP 方法列表（例如 ["GET"]）。
- 其他可选参数：如 response_model、status_code、tags 等，与装饰器方式类似。
#### 如何组织 request handler 函数 
方式一：使用装饰器定义路由的方式中，可直接定义请求处理函数对请求进行处理
```python
@app.get("/")
async def handle_request():
	# ……
	return 
```
方法二：若通过路由组对象调用 `add_api_route`方法，则在第二个参数的位置传递请求处理函数。适合大型项目，可以按功能模块划分路由：
```python
router.add_api_route(
    '/author',
    user_api.get_author_info,
    methods=['get'],
    summary='获取作者信息'
)
```
#### 写一个最简单的 request handler 函数 
方式一：
```python
@app.get("/ping")
async def ping_pong_check() -> str:
	return "pong!"
```
方式二：
```python
router = APIRouter()

router.add_api_route(
	"/ping",
	answer_ping,
	methods=['get'],
	summart="响应 ping 请求"
)

async def answer_ping() -> str:
	return "pong!"
```
#### 如何从 get/post 请求中取出参数 
##### 路径参数（get）：
url 路径中存在的参数，可直接通过在请求处理函数中声明参数进行取用
```python
@app.get("/users/{user_id}") 
async def read_user(user_id: str): 
	return {"user_id": user_id}
```
通过设置函数入参的变量类型，fastapi 可自行校验该变量的类型

###### 预设值：
```python
from enum import Enum
from fastapi import FastAPI
# 通过从 str 继承，API 文档就能把值的类型定义为 字符串，并且能正确渲染
class ModelName(str, Enum): 
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"
    
app = FastAPI()

@app.get("/models/{model_name}")
async def get_model(model_name: ModelName):
    if model_name is ModelName.alexnet:
        return {"model_name": model_name, "message": "Deep Learning FTW!"}

    if model_name.value == "lenet":
        return {"model_name": model_name, "message": "LeCNN all the images"}

    return {"model_name": model_name, "message": "Have some residuals"}
```
使用 Enum 类（`ModelName`）创建使用类型注解的路径参数`model_name`

**枚举元素支持比较**：`if model_name is ModelName.alexnet:`

**获取枚举值**：使用 `model_name.value`获取实际的值

使用 `ModelName.lenet.value` 也能获取值 `"lenet"`

**返回枚举元素**：即使嵌套在 JSON 请求体里（例如， `dict`），也可以从路径操作返回枚举元素。
返回给客户端之前，要把枚举元素转换为对应的值

客户端中的 JSON 响应如下：
```json
{
  "model_name": "alexnet",
  "message": "Deep Learning FTW!"
}
```

###### 包含路径的路径参数
假设_路径操作_的路径为 `/files/{file_path}`。
但需要 `file_path` 中也包含*路径*，比如，`home/johndoe/myfile.txt`。
此时，该文件的 URL 是这样的：`/files/home/johndoe/myfile.txt`。

使用路径转换器可声明包含路径的路径参数，直接使用 Starlette 的选项声明包含*路径*的*路径参数*：
```cmd
/files/{file_path:path}
```
参数名为 `file_path`，结尾部分的`:path` 说明该参数应匹配路径，用法如下：
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/files/{file_path:path}")
async def read_file(file_path: str):
    return {"file_path": file_path}
```

##### 查询参数（get）：
url 末尾含有的参数
声明的参数不是路径参数时，路径操作函数会把该参数自动解释为**查询**参数。
```python
from fastapi import FastAPI

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]

@app.get("/items/")
async def read_item(skip: int = 0, limit: int = 10):
    return fake_items_db[skip : skip + limit]
```
查询字符串是键值对的集合，这些键值对位于 URL 的 `?` 之后，以 `&` 分隔。
```cmd
http://127.0.0.1:8000/items/?skip=0&limit=10
```
这些值都是 URL 的组成部分，因此，它们的类型**本应**是字符串。
但声明 Python 类型（上例中为 `int`）之后，这些值就会转换为声明的类型，并进行类型校验。

将默认值设为`None`即可声明为**可选的**查询参数，如果要把查询参数设置为必选，就不要声明默认值。

##### 请求参数（post）：
客户端向服务端发送的请求体中携带的参数
*路径操作*函数内部可直接访问模型对象的属性：
```python
from fastapi import FastAPI
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

app = FastAPI()

@app.post("/items/")
async def create_item(item: Item):
    item_dict = item.dict()
    if item.tax is not None:
        price_with_tax = item.price + item.tax
        item_dict.update({"price_with_tax": price_with_tax})
    return item_dict
```

> 函数参数按如下规则进行识别：
> - **路径**中声明了相同参数的参数，是路径参数
> - 类型是（`int`、`float`、`str`、`bool` 等）**单类型**的参数，是**查询**参数
> - 类型是 **Pydantic 模型**的参数，是**请求体**

#### 如何定义全局url 拦截函数 
全局 url 拦截函数即中间件，中间件特指在**HTTP 请求到达业务逻辑之前**和**响应返回客户端之前**处理数据的组件，处于“中间”位置，常见用途为：日志记录、用户鉴权、数据校验、错误处理等
##### 1. 官方 demo
创建中间件可在函数的顶部使用装饰器 `@app.middleware("http")`
```python
import time

from fastapi import FastAPI, Request

app = FastAPI()

@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start_time = time.perf_counter()
    response = await call_next(request)
    process_time = time.perf_counter() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response
```
##### 2. 自定义中间件
通过继承`BaseHTTPMiddleware`类并定义`dispatch`方法来实现的。`dispatch`方法是中间件的核心，因为它定义了中间件如何处理进入的请求（`Request`）和如何接收或修改响应（`Response`）。
```python
class PreventCrawlerMiddleware(BaseHTTPMiddleware):
    """ 预防爬虫中间件 """
    NotAllowUAList = [
        "PostmanRuntime",
        "Python"
    ]
    async def dispatch(
            self, request: Request, call_next: RequestResponseEndpoint
    ) -> Response:
        headers = request.headers
        print(headers.get('user-agent'))
        print(headers.get('referer'))
        for not_allow_ua in self.NotAllowUAList:
            if not_allow_ua in headers.get("user-agent"):
                raise Exception("非法请求")
        resp = await call_next(request)
        return resp
```
	
#### 如何获取/修改/存储 cookie,session数据 
##### cookie 操作
###### 获取：
法1：使用 `Cookie()` 作为参数类型注解，告诉 FastAPI 自动从请求的 Cookie 中提取对应的字段信息

```python
from typing import Annotated
from fastapi import Cookie, FastAPI

app = FastAPI()

@app.get("/items/")
async def read_items(ads_id: Annotated[str | None, Cookie()] = None):
    return {"ads_id": ads_id}
```
- `Annotated[str | None, Cookie()]` 是 **FastAPI** 的一种新写法，表示 `ads_id` 参数的类型是 `str` 或 `None`，并且它的值会从请求的 **Cookie** 中获取。
- 如果请求中包含名为 `ads_id` 的 Cookie，FastAPI 会将其值传递给 `ads_id` 参数。如果没有这个 Cookie，`ads_id` 参数将默认为 `None`。

法2：使用 `Request` 对象
```python
from fastapi import Request

@app.get('/')
async def root(request: Request):
    return request.cookies.get('sessionKey')
```
使用 `Request` 对象，提供了更多的灵活性。通过 `request.cookies.get()` 方法手动从 Cookie 中提取特定字段（如 `sessionKey`）。

##### 修改/存储
使用 `set_cookie()` 方法可以在响应中设置或修改 Cookie。该方法允许您指定 Cookie 的名字、值、过期时间、路径等属性。
```python
from fastapi import FastAPI, Response
from datetime import timedelta

app = FastAPI()

@app.get("/set_cookie/")
async def set_cookie(response: Response):
    # 设置一个名为 'sessionKey' 的 Cookie，值为 '123456'
    response.set_cookie("sessionKey", "123456", max_age=timedelta(days=7))  # 设置 Cookie 有效期为 7 天
    return {"message": "Cookie has been set!"}

@app.get("/get_cookie/")
async def get_cookie(sessionKey: str | None = None):
    return {"sessionKey": sessionKey}

@app.get("/delete_cookie/")
async def delete_cookie(response: Response):
    # 删除 Cookie
    response.delete_cookie("sessionKey")
    return {"message": "Cookie has been deleted!"}

```
- **存储 Cookie**：
    - 在 `/set_cookie/` 路由中，我们使用了 `response.set_cookie()` 来设置名为 `sessionKey` 的 Cookie。
    - `max_age=timedelta(days=7)` 设置了该 Cookie 的过期时间为 7 天。你还可以设置 `expires` 或 `path` 等其他属性。
- **获取 Cookie**：
    - 在 `/get_cookie/` 路由中，您可以通过查询参数（如 `sessionKey`）来获取存储在 Cookie 中的值，FastAPI 会自动从请求的 Cookie 中提取该值。
- **删除 Cookie**：
    - 在 `/delete_cookie/` 路由中，我们使用了 `response.delete_cookie()` 来删除 Cookie，传入 Cookie 的名字 `sessionKey` 即可。


##### session 操作
###### 获取
**FastAPI 本身并不直接内置 Session 管理功能**，但是可以利用 Starlette 提供的或社区开发的 Session 中间件。
Starlette（FastAPI 的基础）提供了一个内置的 SessionMiddleware，它使用 **签名的 Cookie (Signed Cookies)** 来存储 Session 数据。这意味着 Session 数据直接存储在客户端的 Cookie 中，而不是服务器端。
```python
from starlette.middleware.sessions import SessionMiddleware # 导入中间件
@app.get("/session/view", summary="获取当前 Session 数据") 
async def get_session_data(request: Request): 
	""" 演示如何获取 Session 数据。 
	中间件在调用此函数前，已经从请求的 Cookie 中加载了 Session 数据 
	并将其放入 `request.session` (如果 Cookie 有效的话)。 
	""" 
	# 使用 .get() 安全地获取值，可以提供默认值 
	username = request.session.get("username", "游客") 
	cart = request.session.get("cart", {}) # 获取购物车，默认为空字典 
	login_time = request.session.get("login_time") 
	print(f"[Get Session] Current session dict:{dict(request.session)}") # 在服务器端打印 Session 内容 
	return JSONResponse({ 
		"message": "当前 Session 数据", 
		"username": username, 
		"cart": cart, 
		"login_time": login_time, 
		"full_session": dict(request.session) # 返回整个 Session 字典的副本 
	})
```
 **获取 Session 信息 (Getting):**
    - 当一个 HTTP 请求到达 FastAPI 应用时，**Session 中间件会率先被执行**。
    - 中间件检查请求中的 Session Cookie。
    - 如果 Cookie 有效（签名正确，未过期等）：
        - 对于 **Cookie 存储型 Session** (如 SessionMiddleware 默认行为): 中间件直接从 Cookie 中解析、验证并解码出 Session 数据。
        - 对于 **服务器端存储型 Session**: 中间件从 Cookie 中获取 Session ID，然后去后端存储（如 Redis）查找对应的 Session 数据。
    - 中间件将加载到的 Session 数据（通常是一个类似字典的对象）**注入到 request 对象中**，最常见的属性名是 request.session。
    - **在你的路径操作函数 (API 路由) 中，你通过访问 request.session 来获取 Session 数据。** 例如：user = request.session.get("user")。

###### 修改/存储
**直接对 request.session 这个对象进行操作**，就像操作一个普通的 Python 字典一样
```python
@app.post("/session/login", summary="模拟登录并修改 Session") 
async def modify_session_login(request: Request, username: str = "default_user"): 
	""" 
	演示如何修改 Session 数据。 直接对 `request.session` 字典进行赋值或更新操作。 
	""" 
	print(f"[Modify Session] Session before login: {dict(request.session)}") 
	# 设置/更新 Session 中的值 
	request.session["username"] = username 
	request.session["logged_in"] = True 
	from datetime import datetime 
	request.session["login_time"] = datetime.now().isoformat() 
	print(f"[Modify Session] Session after login for '{username}': {dict(request.session)}") 
	# *** 隐式存储 *** 
	# 你不需要显式调用 "save" 或 "store"。 
	# 当这个函数返回响应时，SessionMiddleware 会检测到 request.session 已被修改， 
	# 然后它会自动处理存储逻辑： 
	# - 序列化更新后的 session 数据。 
	# - 对数据进行签名。 
	# - 生成包含新数据的 Set-Cookie 响应头，发送给客户端。 
	# 客户端浏览器会自动存储这个新的 Cookie。 🍪 
	return JSONResponse({ 
		"message": f"用户 '{username}' 已登录，Session 已更新。",
		"current_session": dict(request.session) 
	})
```
**存储 Session 信息 :**
- 当路径操作函数执行完毕，准备返回响应时，**Session 中间件会再次介入**。
- 中间件**检查 request.session 对象的状态**，看它是否在请求处理过程中被修改过。
- 如果 Session 数据**被修改了**：
    - 对于 **Cookie 存储型 Session**: 中间件会将更新后的 request.session 字典序列化、签名，并生成一个新的 Set-Cookie 响应头，包含这个新的 Session 数据，随响应一起发送给客户端浏览器。浏览器负责存储这个更新后的 Cookie。
    - 对于 **服务器端存储型 Session**: 中间件会将更新后的 request.session 数据写回到配置的后端存储（如 Redis），覆盖旧的数据。同时，它可能会（如果需要滚动更新 Session ID 或设置过期时间）生成一个包含 Session ID 的 Set-Cookie 响应头发送给客户端。
- 如果 Session 数据**未被修改**，中间件通常不会执行存储操作，也不会发送不必要的 Set-Cookie 头（除非需要更新过期时间等）

**总结要点 ✨:**
- **中间件是核心:** Session 的加载（读）和持久化（写）这两个关键步骤是由中间件自动完成的。
- **request.session 是接口:** 你的应用程序代码通过 request.session 这个由中间件提供的接口来与 Session 数据交互（获取和修改）。
- **修改触发存储:** 你在路由中对 request.session 的修改，会**标记**这个 Session 为“已更改”，从而触发中间件在响应阶段执行存储逻辑。
- **存储方式依赖中间件配置:** 数据最终是存储在客户端 Cookie 里还是服务器后端（Redis、数据库等），取决于你选择和配置了哪种 Session 中间件

#### 如何修改/输出 http header 数据 
##### 官方推荐方式：
使用和 `Path`、`Query`、`Cookie` 一样的结构定义 header 参数。
第一个值是默认值，还可以传递所有验证参数或注释参数
```python
from typing import Annotated

from fastapi import FastAPI, Header

app = FastAPI()

@app.get("/items/")
async def read_items(user_agent: Annotated[str | None, Header()] = None):
    return {"User-Agent": user_agent}
```

> **说明**：必须使用 `Header` 声明 header 参数，否则该参数会被解释为查询参数。

使用 `JSONResponse`设置响应头
```python
from typing import Annotated
from fastapi import FastAPI, Header
from fastapi.responses import JSONResponse

app = FastAPI()

@app.get("/items/")
async def read_items(user_agent: Annotated[str | None, Header()] = None):
    # 定义响应内容
    content = {"User-Agent": user_agent}
    # 设置自定义响应头
    headers = {
        "X-Custom-Header": "CustomValue",
        "X-Another-Header": "AnotherValue"
    }
    # 返回 JSONResponse，包含自定义头
    return JSONResponse(content=content, headers=headers)
```

##### 直接使用`Response` 对象
FastAPI 提供了 `Request` 对象，可以用来访问客户端发送的请求头
```python
from fastapi import FastAPI, Request

app = FastAPI()

@app.get("/get-headers")
async def get_headers(request: Request):
    headers = dict(request.headers)  # 获取所有请求头
    return {"headers": headers}
```
**说明**：
- `request.headers` 是一个类似字典的对象，包含所有客户端发送的头信息（如 `User-Agent`、`Accept` 等）。
- 你可以直接返回 `headers` 或从中提取特定头，例如 `request.headers.get("user-agent")`。

```python
from typing import Annotated
from fastapi import FastAPI, Header, Response

app = FastAPI()

@app.get("/items/")
async def read_items(user_agent: Annotated[str | None, Header()] = None, response: Response):
    # 设置响应头
    response.headers["X-Custom-Header"] = "CustomValue"
    response.headers["X-Another-Header"] = "AnotherValue"
    # 返回响应内容
    return {"User-Agent": user_agent}
```
**说明**：
- **response: Response**：通过依赖注入获取 Response 对象。
- **response.headers**：直接修改响应头的字典，添加或更新头信息。
- FastAPI 会自动将返回值（字典）转为 JSON 响应，并保留设置的头。

#### 如何部署app程序 
通过 Nginx 进行部署
#### 如何配置开发环境 
配置 python3 环境
安装 fastapi 库：
```python
# 附带一些默认的可选标准依赖项
pip install fastapi[standard]
# 若不想安装可选依赖，则
pip install fastapi
```
#### 如何配置静态文件访问 
FastAPI 提供了 `StaticFiles` 类来处理静态文件的路由。只需要将静态文件目录与一个 URL 路径绑定，然后通过这个路径就可以访问该目录中的文件。
步骤：
- **导入 `StaticFiles` 类**：首先，你需要导入 FastAPI 和 `StaticFiles` 类。
- **指定静态文件目录**：定义静态文件所在的目录（如 `static` 目录）。
- **将 `StaticFiles` 与 FastAPI 路由关联**：通过 `app.mount()` 方法将静态文件目录映射到一个 URL 路径。

```python
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles

app = FastAPI()

# 将 static 目录挂载到 /static 路径
app.mount("/static", StaticFiles(directory="static"), name="static")

# 其他路由
@app.get("/")
async def read_root():
    return {"message": "Hello, FastAPI!"}

```

#### 如何访问数据库 
##### ORM
Fastapi 官方推荐 SQLAlchemy ORM 库，其与 Fastapi 的核心特性（Pydantic）结合非常好。具体用法见下一个问题。
##### 查询构建器

##### 原生数据库驱动
python 提供了很多数据库驱动程序，可直接使用驱动程序连接数据库，手写 SQL 语句。以 mysql 为例：
需安装 mysql 数据库驱动程序：
```python
pip install mysql-connector-python pymysql
```
连接数据库示例：
db.py
```python
from fastapi import Depends 
import mysql.connector 

def get_db_connection(): 
	connection = mysql.connector.connect( 
		host='localhost', 
		port=3306, 
		user="root", 
		password="123456", 
		database="example_db" 
	) 
	return connection 
def get_db(): 
	connection = get_db_connection() 
	db = connection.cursor() 
	try: 
		yield db 
	finally: 
		db.close() 
		connection.close()

```
db_router.py
```python
from fastapi import FastAPI, Depends 
from mysql.connector import cursor 
from db import get_db 
import json 

app = FastAPI() 

# def get_db(db: cursor.MySQLCursor = Depends(get_db)): 
# return db 

@app.get("/users/") 
async def get_users(db: cursor.MySQLCursor = Depends(get_db)): 
	query = "SELECT * FROM users" 
	db.execute(query) 
	result = db.fetchall() 
	if result: 
		return {"users": result} 
	else: 
		return {"error": "User not found"} 

@app.get("/users/{user_id}") 
async def get_user(user_id: int, db: cursor.MySQLCursor = Depends(get_db)): 
	query = "SELECT * FROM users WHERE id = %s" 
	db.execute(query, (user_id,)) 
	result = db.fetchall() 
	if result: 
		return {"user_id": result[0][0], "username": result[0][1]} 
	else: 
		return {"error": "User not found"} 

@app.get("/user_name/{user_name}") 
async def insert_user(user_name: str, db: cursor.MySQLCursor = Depends(get_db)): 
	query = "INSERT INTO users (name) VALUES (%s)" 
	db.execute(query, (user_name,)) 
	result = db.fetchone() 
	db.execute("COMMIT") 
	return {"user_name": user_name}

```

#### 是否支持ORM 
支持，使用 SQLModel ORM

#### 如何维护表结构的变更 
结合 SQLAlchemy 和 Alembic，开发者可以通过模型定义表结构，并使用 Alembic 自动生成迁移脚本
```python
from sqlalchemy import Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True)
    name = Column(String)
```
Alembic 命令：
```bash
alembic revision --autogenerate -m "Add user table"
alembic upgrade head
```


#### 如何定义/组织/初始化 数据表 
##### 导入 `SQLModel`创建数据库模型
```python
from typing import Annotated

from fastapi import Depends, FastAPI, HTTPException, Query
from sqlmodel import Field, Session, SQLModel, create_engine, select

class Hero(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    name: str = Field(index=True)
    age: int | None = Field(default=None, index=True)
    secret_name: str

# Code below omitted 👇
```
##### 创建引擎
SQLModel 的引擎 `engine`（实际上它是一个 SQLAlchemy `engine` ）是用来与数据库**保持连接**的。
只需构建**一个 `engine`**，来让您的所有代码连接到同一个数据库。
```python
# Code above omitted 👆

sqlite_file_name = "database.db"
sqlite_url = f"sqlite:///{sqlite_file_name}"

connect_args = {"check_same_thread": False}
engine = create_engine(sqlite_url, connect_args=connect_args)

# Code below omitted 👇
```
使用 `check_same_thread=False` 可以让 FastAPI 在不同线程中使用同一个 SQLite 数据库。这很有必要，因为**单个请求**可能会使用**多个线程**（例如在依赖项中）。

不用担心，我们会按照代码结构确保**每个请求使用一个单独的 SQLModel _会话_**，这实际上就是 `check_same_thread` 想要实现的。

##### 创建表
添加一个函数，使用 `SQLModel.metadata.create_all(engine)` 为所有*表模型***创建表**。
```python
# Code above omitted 👆

def create_db_and_tables():
    SQLModel.metadata.create_all(engine)

# Code below omitted 👇
```
##### 启动时创建数据库表
在应用程序启动时创建数据库表
```python
# Code above omitted 👆

app = FastAPI()

@app.on_event("startup")
def on_startup():
    create_db_and_tables()

# Code below omitted 👇
```

#### 如何对接orm系统和现有的表结构 
SQLAlchemy + Alembic 同时支持**表结构变更**和**对接orm系统与现有表结构**；SQLModel 更适合变更，对接需手动映射。
#### 掌握最基本的add/delete/按字段查询/count/slice/order by 
创建 SessionDep 类型别名
```python
def get_session(): 
	with Session(engine) as session: 
		yield session 
		
SessionDep = Annotated[Session, Depends(get_session)]
```
- 创建类型别名 SessionDep ，表示它是一个依赖注入的数据库会话类型
- 在 FastAPI 中，可以通过将这个 `SessionDep` 用作参数类型来自动注入数据库会话。例如，FastAPI 会在处理请求时自动调用 `get_session` 函数来提供一个数据库会话。
##### add
```python
# Code above omitted 👆

@app.post("/heroes/")
def create_hero(hero: Hero, session: SessionDep) -> Hero:
    session.add(hero)
    session.commit()
    session.refresh(hero)
    return hero

# Code below omitted 👇
```
##### delete
```python
# Code above omitted 👆

@app.delete("/heroes/{hero_id}")
def delete_hero(hero_id: int, session: SessionDep):
    hero = session.get(Hero, hero_id)
    if not hero:
        raise HTTPException(status_code=404, detail="Hero not found")
    session.delete(hero)
    session.commit()
    return {"ok": True}
```
##### 按字段查询
```python
# Code above omitted 👆

@app.get("/heroes/{hero_id}")
def read_hero(hero_id: int, session: SessionDep) -> Hero:
    hero = session.get(Hero, hero_id)
    if not hero:
        raise HTTPException(status_code=404, detail="Hero not found")
    return hero

@app.get("/heroes/") 
def read_heroes( session: SessionDep, offset: int = 0, limit: Annotated[int, Query(le=100)] = 100, ) -> list[Hero]: 
	heroes = session.exec(select(Hero).offset(offset).limit(limit)).all() 
	return heroes

# Code below omitted 👇
```
- **count** ：`count()` 方法用于计算查询结果的数量，即返回符合查询条件的记录数
		**用途**：获取查询结果的条目数量
```python
# 查询符合条件的记录数
result_count = session.query(User).filter(User.age > 30).count()
print(result_count)  # 输出年龄大于30的用户数量
```
- **slice** ：`slice()` 方法用于从查询结果中提取一个切片，类似于 Python 中的列表切片。
		**用途**：对查询结果进行分页或限制结果的范围。
```python
# 获取前10条数据
users_slice = session.query(User).filter(User.age > 30).slice(0, 10).all()
print(users_slice)
```
`slice(start, stop)` 可以用于指定返回结果的起始位置和结束位置。注意，`slice` 在 SQLAlchemy 中的实现是基于 `LIMIT` 和 `OFFSET`
> 注意：`slice()` 实际上是通过 `limit()` 和 `offset()` 方法实现的，因此你也可以用这两个方法替代 `slice()`。

- **order by**：`order_by()` 方法用于对查询结果进行排序。你可以根据一个或多个列进行升序或降序排序
		**用途**：对查询结果进行排序，默认是升序排序。如果要降序排序，可以使用 `desc()`。
```python 
# 按照年龄升序排序
users = session.query(User).order_by(User.age).all()
print(users)

# 按照年龄降序排序
from sqlalchemy import desc
users = session.query(User).order_by(desc(User.age)).all()
print(users)

```

##### 改动
单个对象属性改动
```python
@app.get("/heroes/{hero_id}")
def modify_hero(hero_id: int, session: SessionDep) -> Hero:
	hero = session.get(Hero, hero_id)
	if not hero:
		raise HTTPException(status_code=404, detail="Hero not found")
	hero.name = "jerry"
	hero.age = 18
	hero.secret_name = "g"
	session.commit()
	session.refresh(hero)
```
调用 `update()` 方法
```python
@app.put("/heroes/{hero_id}")
def modify_hero(hero_id: int, session: SessionDep) -> Hero:
    # 批量更新，而不需要加载对象
    rows_updated = session.query(Hero).filter(Hero.id == hero_id).update({
        Hero.name: "jerry",
        Hero.age: 18,
        Hero.secret_name: "g"
    })
    
    if rows_updated == 0:
        raise HTTPException(status_code=404, detail="Hero not found")
    
    session.commit()
    
    # 重新查询获取更新后的数据
    hero = session.get(Hero, hero_id)
    
    return hero

```
#### 如何直接使用sql 访问数据库 不支持orm (这样的web框架，不用也罢) 
可以，python 提供了众多数据库的驱动，直接导入对应数据库的驱动库，然后使用手写 sql 的方式也可以进行对数据库的访问。
#### 如何通过 http get/post 获取远程数据 
##### 使用 requests 库
**requests**：同步库，代码简单，适合快速开发或低并发场景。缺点是阻塞主线程，可能在高并发场景下性能较差。
```python
from fastapi import FastAPI
import requests

app = FastAPI()

# GET 请求：从远程 API 获取数据
@app.get("/fetch-data")
def fetch_data():
    response = requests.get("https://api.example.com/users")
    if response.status_code == 200:
        return response.json()  # 返回远程 API 的 JSON 数据
    return {"error": f"Failed to fetch data, status code: {response.status_code}"}

# POST 请求：向远程 API 发送数据并获取响应
@app.post("/submit-and-fetch")
def submit_and_fetch(data: dict):  # 接受 JSON 数据
    response = requests.post("https://api.example.com/login", json=data)
    if response.status_code == 200:
        return response.json()  # 返回远程 API 的 JSON 数据
    return {"error": f"Failed to submit data, status code: {response.status_code}"}
```
##### 使用 httpx 库
**httpx**：支持异步（async/await），与 FastAPI 的异步特性更契合，适合高并发场景。API 设计与 requests 类似，易于切换。
```python
from fastapi import FastAPI
import httpx

app = FastAPI()

# GET 请求：从远程 API 获取数据
@app.get("/fetch-data")
async def fetch_data():
    async with httpx.AsyncClient() as client:
        response = await client.get("https://api.example.com/users")
        if response.status_code == 200:
            return response.json()  # 返回远程 API 的 JSON 数据
        return {"error": f"Failed to fetch data, status code: {response.status_code}"}

# POST 请求：向远程 API 发送数据并获取响应
@app.post("/submit-and-fetch")
async def submit_and_fetch(data: dict):  # 接受 JSON 数据
    async with httpx.AsyncClient() as client:
        response = await client.post("https://api.example.com/login", json=data)
        if response.status_code == 200:
            return response.json()  # 返回远程 API 的 JSON 数据
        return {"error": f"Failed to submit data, status code: {response.status_code}"}
```
#### 如何 parse json 
##### pydantic 模型来解析 json
FastAPI 自动解析 JSON 请求体，并将其映射到 Pydantic 模型实例
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

# 定义 Pydantic 模型
class Item(BaseModel):
    name: str
    description: str
    price: float
    tax: float = None

# 路由接收 JSON 数据并解析
@app.post("/items/")
async def create_item(item: Item):
    return {"name": item.name, "price": item.price}
```
FastAPI 在解析 JSON 数据时会自动验证数据结构是否符合定义的 Pydantic 模型。若数据格式不正确，FastAPI 会自动返回 422 错误，并提供详细的错误信息。
##### 手动解析 json
使用 `request.json()` 方法手动解析 JSON 数据
```python
from fastapi import FastAPI, Request

app = FastAPI()

@app.post("/manual_json/")
async def manual_json(request: Request):
    # 手动解析 JSON 数据
    data = await request.json()
    # 获取解析后的数据
    name = data.get("name")
    price = data.get("price")
    return {"name": name, "price": price}
```
**解释：**
- `request.json()` 方法会解析请求体中的 JSON 数据并将其转换为 Python 字典。
- 然后，你可以像操作普通字典一样处理数据，使用 `data.get("name")` 获取字段值。
#### 如何 parse xml 
##### 解析 xml 请求
使用 Python 内置的 `xml.etree.ElementTree` 
```python
from fastapi import FastAPI, Request
import xml.etree.ElementTree as ET

app = FastAPI()

@app.post("/parse_xml")
async def parse_xml(request: Request):
    # 获取原始的 XML 数据
    body = await request.body()
    # 将 XML 数据解析成 ElementTree 对象
    try:
        tree = ET.ElementTree(ET.fromstring(body))
        root = tree.getroot()
        # 提取 XML 中的特定信息
        result = {child.tag: child.text for child in root}
        return result
    except ET.ParseError as e:
        return {"error": "Invalid XML format", "details": str(e)}
```
第三方库 `lxml` 来解析 XML 数据
```python
from fastapi import FastAPI, Request
from lxml import etree

app = FastAPI()

@app.post("/parse_xml_lxml")
async def parse_xml_lxml(request: Request):
    # 获取原始的 XML 数据
    body = await request.body()
    try:
        # 使用 lxml 来解析 XML
        root = etree.fromstring(body)
        # 提取 XML 中的特定信息
        result = {child.tag: child.text for child in root}
        return result
    except etree.XMLSyntaxError as e:
        return {"error": "Invalid XML format", "details": str(e)}
```
使用`XPath`提取特定节点的数据或者手动遍历 xml 节点

##### 返回 xml 响应
FastAPI 默认返回的是 JSON 格式，如果你需要返回 XML 格式，可以使用 `XMLResponse` 来设置响应类型为 `application/xml`
```python
from fastapi import FastAPI
from fastapi.responses import XMLResponse

app = FastAPI()

@app.get("/get_xml")
async def get_xml():
    xml_data = """
    <response>
        <status>success</status>
        <message>XML response from FastAPI</message>
    </response>
    """
    return XMLResponse(content=xml_data)
```
也可以通过 `Response` 方法将 `media_type` 设置为 `application/xml` 来返回 XML 响应：
```python
from fastapi import FastAPI, Response

app = FastAPI()

@app.get("/xml/")
def get_legacy_data():
    data = """<?xml version="1.0"?>
    <shampoo>
        <Header>
            Apply shampoo here.
        </Header>
        <Body>
            You'll have to use soap here.
        </Body>
    </shampoo>
    """
    return Response(content=data, media_type="application/xml")
```
#### 如何输出为 json 
- FastAPI 默认会将响应数据自动转换为 JSON 格式，只要返回的是 Python 字典、列表等数据结构。
```python
@app.get("/json_response")
async def json_response():
    return {"message": "This is a JSON response", "status": "success"}
```

- 可以显式使用 `JSONResponse` 来返回自定义的 JSON 响应，设置自定义的 HTTP 状态码、头部等。
```python
from fastapi import FastAPI
from fastapi.responses import JSONResponse

app = FastAPI()

@app.get("/custom_json_response")
async def custom_json_response():
    content = {"message": "This is a custom JSON response", "status": "success"}
    return JSONResponse(content=content, status_code=201)
```

- 可以使用 Pydantic 模型作为响应模型，来定义和验证返回数据的格式和类型。
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

# 定义响应模型
class ItemResponse(BaseModel):
    message: str
    status: str

@app.get("/json_with_model", response_model=ItemResponse)
async def json_with_model():
    return ItemResponse(message="This is a JSON response with a model", status="success")
```

- 即使是错误响应，FastAPI 也会以 JSON 格式返回详细的错误信息。
```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

@app.get("/trigger_404")
async def trigger_404():
    raise HTTPException(status_code=404, detail="Item not found")
```

#### 如何处理状态码:404和50x 
FastAPI 中处理 HTTP 状态码（例如 404 和 50x 状态码）通常有两种常见方式：一是通过 FastAPI 的内置异常处理机制，二是通过自定义的异常处理器来捕获特定的错误并返回合适的响应。
##### 一：通过 FastAPI 内置异常处理机制对错误进行处理
FastAPI 会自动处理未定义的路由，并返回一个 404 错误：
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def read_root():
    return {"message": "Welcome to FastAPI!"}

# 没有定义的路由会自动返回 404 错误

```
访问未定义的路径时，响应结果：
```python
{
  "detail": "Not Found"
}
```
FastAPI 会自动处理服务器发生的异常，返回一个 500 错误响应。例如，如果你的应用抛出一个没有捕获的异常，FastAPI 会自动返回：
```python
{
  "detail": "Internal Server Error"
}
```

##### 二：自定义异常处理器捕获错误响应
自定义错误处理：
```python
from fastapi import FastAPI, HTTPException, Request
from fastapi.responses import JSONResponse
from starlette.exceptions import HTTPException as StarletteHTTPException
import logging

# 创建 FastAPI 实例
app = FastAPI()

# 创建 logger 用于记录错误
logger = logging.getLogger("uvicorn")
logging.basicConfig(level=logging.INFO)

# 404 错误处理
@app.exception_handler(StarletteHTTPException)
async def http_exception_handler(request: Request, exc: StarletteHTTPException):
    if exc.status_code == 404:
        return JSONResponse(
            status_code=404,
            content={"message": "Resource not found. Please check the URL."}
        )
    return await request.app.default_exception_handler(request, exc)

# 500 错误处理
@app.exception_handler(Exception)
async def server_error_handler(request: Request, exc: Exception):
    logger.error(f"Internal server error: {exc}")  # 记录错误信息
    return JSONResponse(
        status_code=500,
        content={"message": "Oops! Something went wrong on our end. Please try again later."}
    )

# 示例路由
@app.get("/")
async def read_root():
    return {"message": "Welcome to FastAPI!"}

@app.get("/error")
async def create_error():
    raise Exception("This is a test of a 500 error.")

# 404 错误示例
# 当访问不存在的路由时，例如 `/non_existent_path`，会触发 404 错误

```

##### 使用 `status_code`参数来手动设置状态码
手动设置状态码，可以使用 FastAPI 路由的 `status_code` 参数
```python
from fastapi import FastAPI, HTTPException, status

app = FastAPI()

# 触发 404 错误
@app.get("/not_found")
async def not_found():
    raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Resource not found.")

# 触发 500 错误
@app.get("/server_error")
async def server_error():
    raise HTTPException(status_code=status.HTTP_500_INTERNAL_SERVER_ERROR, detail="Internal server error.")
```
#### 如何处理文件上传
##### 可选单个文件上传
```python
from typing import Annotated

from fastapi import FastAPI, File, UploadFile

app = FastAPI()


@app.post("/files/")
async def create_file(file: Annotated[bytes | None, File()] = None):
    if not file:
        return {"message": "No file sent"}
    else:
        return {"file_size": len(file)}


@app.post("/uploadfile/")
async def create_upload_file(file: UploadFile | None = None):
    if not file:
        return {"message": "No upload file sent"}
    else:
        return {"filename": file.filename}
```
##### 多文件上传
FastAPI 支持同时上传多个文件。
可用同一个「表单字段」发送含多个文件的「表单数据」。
上传多个文件时，要声明含 `bytes` 或 `UploadFile` 的列表（`List`）：
```python
from typing import Annotated

from fastapi import FastAPI, File, UploadFile
from fastapi.responses import HTMLResponse

app = FastAPI()

@app.post("/files/")
async def create_files(files: Annotated[list[bytes], File()]):
    return {"file_sizes": [len(file) for file in files]}

@app.post("/uploadfiles/")
async def create_upload_files(files: list[UploadFile]):
    return {"filenames": [file.filename for file in files]}

@app.get("/")
async def main():
    content = """
<body>
<form action="/files/" enctype="multipart/form-data" method="post">
<input name="files" type="file" multiple>
<input type="submit">
</form>
<form action="/uploadfiles/" enctype="multipart/form-data" method="post">
<input name="files" type="file" multiple>
<input type="submit">
</form>
</body>
    """
    return HTMLResponse(content=content)
```

#### 如何进行单元测试
##### 推荐使用 pytest 进行测试
pytest：非常流行的第三方测试框架，比 `unittest` 更加简洁和灵活，支持自动化发现测试用例、丰富的断言机制、插件系统等。功能强大，可以处理多种类型的测试用例。它支持各种测试策略，如多个依赖模块之间的测试。
**单元测试：**
```python
# test_calculator.py
def add(a, b):
    return a + b

def test_add():
    assert add(1, 2) == 3
    assert add(-1, 1) == 0
```
运行：
```python
pytest test_calculator.py
```

**集成测试：**
```python
import requests

def test_user_creation():
    # 假设你的后端 API 创建用户
    response = requests.post("http://localhost:8000/api/users", json={"name": "Alice"})
    assert response.status_code == 201
    assert response.json()["name"] == "Alice"
```
