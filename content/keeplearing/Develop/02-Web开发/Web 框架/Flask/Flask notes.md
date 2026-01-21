#### 最小 Flask 应用：
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'

```
**解释：**
1. 首先我们导入了 [`Flask`](https://flask.github.net.cn/api.html#flask.Flask "flask.Flask") 类。 该类的实例将会成为我们的 WSGI 应用。
2. 接着我们创建一个该类的实例。第一个参数是应用模块或者包的名称。如果你使用 一个单一模块（就像本例），那么应当使用 `__name__` ，因为名称会根据这个 模块是按应用方式使用还是作为一个模块导入而发生变化（可能是 ‘__main__’ ， 也可能是实际导入的名称）。这个参数是必需的，这样 Flask 才能知道在哪里可以 找到模板和静态文件等东西。更多内容详见 [`Flask`](https://flask.github.net.cn/api.html#flask.Flask "flask.Flask") 文档。
3. 然后我们使用 [`route()`](https://flask.github.net.cn/api.html#flask.Flask.route "flask.Flask.route") 装饰器来告诉 Flask 触发函数的 URL 。
4. 函数名称被用于生成相关联的 URL 。函数最后返回需要在用户浏览器中显示的信息。

把它保存为 `hello.py` 或其他类似名称。请不要使用 `flask.py` 作为应用名称，这会与 Flask 本身发生冲突。
**运行方式：**
1. 使用 python 直接运行（需要添加`app.run()`）
```python
if __name__ == '__main__':
    app.run()
```
然后直接运行：
```cmd
python hello.py
```
2. 使用命令行参数（Flask 2.0+）
```bash
flask --app hello run
```
#### 如何定义 url router 
##### 1. 使用 `route()`装饰器将函数绑定到 URL
最简单直观，适合大多数场景：
```python
@app.route('/')
def index():
    return 'Index Page'

@app.route('/hello')
def hello():
    return 'Hello, World'
```
##### 2. 使用 `add_url_rule` 方法
编程式添加路由，适合动态路由注册
```python
def index():
    return 'Index Page'
    
app.add_url_rule('/', 'index', index)
```
解释：
- 参数一：URL 规则，即访问路径
- 参数二：端点名称，用于 URL 反向生成
- 参数三：视图函数，处理该 URL 请求的函数
**URL 反向生成**：
将"/"路径绑定至“index”上，在其他任何地方使用 “index”生成 url 时，会进行自动替换，相当于C语言中的宏定义，但更加灵活和动态。
- **动态性**
```python
# 如果将来修改路由
app.add_url_rule('/home', 'index', index)  # 改成 /home
# 所有使用 url_for('index') 的地方会自动更新，不需要重新编译
```
- **支持参数**
```python
@app.route('/user/<int:user_id>')
def user_profile(user_id):
    return f'用户资料: {user_id}'

# 可以动态传参
profile_url = url_for('user_profile', user_id=123)  # 生成 /user/123
```
`url_for()` 的第一个参数是视图函数的名称（也就是端点名称），这里使用 `user_profile`是因为：当使用装饰器`@app.route` 定义路由时，如果没有明确指定端点名称，Flask 会自动使用视图函数的名称作为端点名称。当然，端点名称也可以使用参数`endpoint`来指定：
```python
# 示例2：显式指定端点名称
@app.route('/user/<int:user_id>', endpoint='get_user')
def user_profile(user_id):
    return f'用户资料: {user_id}'
# url_for('get_user', user_id=123) 使用指定的端点名称
```
- **支持额外参数**
```python
# 可以添加查询参数
url = url_for('index', page=2, sort='name')  # 生成 /?page=2&sort=name
```
##### 3. 使用 `Blueprint`进行模块化路由
适合大型应用的模块化开发
```python
from flask import Blueprint

# 创建一个 Blueprint 对象
admin = Blueprint('admin', __name__, url_prefix='/admin')

@admin.route('/')
def admin_index():
    return 'Admin Index Page'

# 在主应用中注册 blueprint
app.register_blueprint(admin)
```
##### 4. 动态路由
处理带参数的 URL
```python
@app.route('/user/<username>')
def show_user(username):
    return f'User: {username}'

@app.route('/post/<int:post_id>')
def show_post(post_id):
    return f'Post: {post_id}'
```
##### 5. HTTP 方法路由
处理不同的 HTTP 请求方法
```python
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        return do_login()
    else:
        return show_login_form()
```
#### 如何组织 request handler 函数 
##### 1. 直接在应用文件中定义
小型应用或原型开发：
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    return 'Index Page'
    
@app.route('/user')
def user():
    return 'User Page'
```
##### 2. 使用 Blueprint 进行模块化组织
适合大型应用，按功能模块划分：
```python
# blueprints/admin.py
from flask import Blueprint

admin = Blueprint('admin', __name__)

@admin.route('/dashboard')
def dashboard():
    return 'Admin Dashboard'

# blueprints/user.py
from flask import Blueprint

user = Blueprint('user', __name__)

@user.route('/profile')
def profile():
    return 'User Profile'

# app.py
from flask import Flask
from blueprints.admin import admin
from blueprints.user import user

app = Flask(__name__)
app.register_blueprint(admin, url_prefix='/admin')
app.register_blueprint(user, url_prefix='/user')
```

##### 3. 使用类视图
适合 RESTful API 或复杂的视图逻辑：
```python
from flask import Flask
from flask.views import MethodView

app = Flask(__name__)

class UserAPI(MethodView):
    def get(self, user_id):
        # 处理 GET 请求
        return f'Get user {user_id}'
    def post(self):
        # 处理 POST 请求
        return 'Create user'
    def put(self, user_id):
        # 处理 PUT 请求
        return f'Update user {user_id}'
    def delete(self, user_id):
        # 处理 DELETE 请求
        return f'Delete user {user_id}'
# 注册视图
user_view = UserAPI.as_view('user_api')
app.add_url_rule('/users/', view_func=user_view, methods=['GET', 'POST'])
app.add_url_rule('/users/<int:user_id>', view_func=user_view, methods=['GET', 'PUT', 'DELETE'])
```

#### 写一个最简单的request handler 函数 【见上一个问题】
#### 如何从 get/post 请求中取出参数 
##### 1. URL 路径参数
通过把 URL 的一部分标记为 `<variable_name>` 就可以在 URL 中添加变量。标记的 部分会作为关键字参数传递给函数。通过使用 `<converter:variable_name>` ，可以 选择性的加上一个转换器，为变量指定规则。
```python
@app.route('/user/<username>')
def show_user_profile(username):
    # show the user profile for that user
    return 'User %s' % escape(username)

@app.route('/post/<int:post_id>')
def show_post(post_id):
    # show the post with the given id, the id is an integer
    return 'Post %d' % post_id

@app.route('/path/<path:subpath>')
def show_subpath(subpath):
    # show the subpath after /path/
    return 'Subpath %s' % escape(subpath)
```
转换器类型：

| `string` | （缺省值） 接受任何不包含斜杠的文本   |
| -------- | -------------------- |
| `int`    | 接收正整数                |
| `float`  | 接收正浮点数               |
| `path`   | 类似 `string` ，但可以包含斜杠 |
| `uuid`   | 接受 UUID 字符串          |
##### 2. GET 请求参数(Query String)
使用 `request.args`获取 URL 中的查询参数:
```python
from flask import request

@app.route('/search')
def search():
    # /search?q=python&page=2
    query = request.args.get('q')  # 获取单个参数
    page = request.args.get('page', 1, type=int)  # 带默认值和类型转换
    # 获取所有参数
    all_args = request.args
    return f'Search for: {query}, Page: {page}'
```
##### 3. POST 请求参数
处理不同类型的 POST 数据：
```python
from flask import request

@app.route('/submit', methods=['POST'])
def submit():
    # 处理 form-data 格式数据
    username = request.form.get('username')
    password = request.form.get('password')
    # 处理 JSON 格式数据
    json_data = request.get_json()
    # 处理文件上传
    file = request.files.get('file')
    # 获取原始数据
    raw_data = request.data
    return 'Data received'
```


#### 如何定义全局url 拦截函数 
使用装饰器定义全局 URL 拦截函数（中间件）
##### 1. 请求前拦截`before_request`
```python
from flask import Flask, g, request

app = Flask(__name__)

@app.before_request
def before_request():
    # 检查用户是否登录
    if 'user_id' not in session and request.endpoint != 'login':
        return redirect(url_for('login'))
    # 可以在 g 对象中存储数据，供后续视图函数使用
    g.user = get_current_user()
```
##### 2. 请求后拦截`after_request`
在每个请求处理后执行，可以修改响应：
```python
@app.after_request
def after_request(response):
    # 添加通用响应头
    response.headers['Server'] = 'MyServer'
    response.headers['X-Content-Type-Options'] = 'nosniff'
    return response
```
##### 3. 针对特定 URL 的拦截器
```python
from functools import wraps

def login_required(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if not g.user:
            return redirect(url_for('login'))
        return f(*args, **kwargs)
    return decorated_function

@app.route('/admin')
@login_required
def admin():
    return '管理员页面'
```
##### 4. 错误处理拦截
```python
@app.errorhandler(404)
def page_not_found(error):
    return '页面未找到', 404

@app.errorhandler(500)
def internal_error(error):
    return '服务器错误', 500
```
##### 5. 请求结束后的清理工作
```python
@app.teardown_request
def teardown_request(exception):
    db = getattr(g, 'db', None)
    if db is not None:
        db.close()
```
##### 6. 蓝图级别的拦截器
在特定蓝图中注册拦截器：
```python
admin = Blueprint('admin', __name__)

@admin.before_request
def before_request():
    if not current_user.is_admin:
        abort(403)
```

> 装饰器的执行顺序是从下到上的，所以在设计多个装饰器时要注意它们的顺序。
```python
@app.route('/admin/users')
@login_required        # 第二个执行
@admin_required       # 第一个执行
def manage_users():
    return '用户管理页面'
```

#### 如何获取/修改/存储 cookie,session数据 
##### Cookie
获取 Cookie：
```python
from flask import request

@app.route('/api')
def api():
    # 获取单个 Cookie
    user_id = request.cookies.get('user_id')
    # 获取所有 Cookie
    all_cookies = request.cookies
    return f'Cookie值：{user_id}'
```
设置 Cookie：
```python
from flask import make_response

@app.route('/set_cookie')
def set_cookie():
    resp = make_response('Cookie已设置')
    # 设置基本cookie
    resp.set_cookie('username', 'admin')
    # 设置带参数的cookie
    resp.set_cookie('user_id', '123', 
        max_age=3600,           # 过期时间(秒)
        expires=None,           # 具体过期时间
        path='/',              # Cookie路径
        domain=None,           # Cookie域名
        secure=False,          # 是否只通过HTTPS传输
        httponly=True,         # 是否只允许HTTP访问
        samesite=None          # Cookie的SameSite属性
    )
    return resp
```
删除 Cookie：
```python
@app.route('/delete_cookie')
def delete_cookie():
    resp = make_response('Cookie已删除')
    resp.delete_cookie('username')  # 删除指定的cookie
    return resp
```
实际应示例：
```python
@app.route('/login', methods=['POST'])
def login():
    if check_login_credentials():  # 验证登录信息
        resp = make_response('登录成功')
        # 设置登录cookie
        resp.set_cookie('user_id', '123', 
            max_age=3600,      # 1小时后过期
            httponly=True      # 防止XSS攻击
        )
        return resp
    return '登录失败'

@app.route('/logout')
def logout():
    resp = make_response('已退出登录')
    # 删除登录cookie
    resp.delete_cookie('user_id')
    return resp
```
注意事项：
1. Cookie 大小有限制（通常为4KB）
2. 敏感信息建议使用 Session 而不是 Cookie
3. 设置 `httponly=True` 可以防止 JavaScript 访问 Cookie，增加安全性
4. 使用 `secure=True` 确保 Cookie 只通过 HTTPS 传输

##### Session
除了请求对象之外还有一种称为 [`session`](https://flask.github.net.cn/api.html#flask.session "flask.session") 的对象，允许你在不同请求 之间储存信息。这个对象相当于用密钥签名加密的 cookie ，即用户可以查看你的 cookie ，但是如果没有密钥就无法修改它。
配置 Session：
```python
from flask import Flask, session

app = Flask(__name__)
# 设置密钥，用于加密 session 数据
app.secret_key = 'your-secret-key'  # 建议使用随机生成的复杂字符串
```
存储 Session 数据：
```python
@app.route('/login', methods=['POST'])
def login():
    if check_credentials():  # 验证用户信息
        # 存储数据到 session
        session['user_id'] = 123
        session['username'] = 'admin'
        session['is_logged'] = True
        return '登录成功'
    return '登录失败'
```
获取 Session 数据：
```python
@app.route('/profile')
def profile():
    # 获取单个值
    user_id = session.get('user_id')
    # 获取值，如果不存在返回默认值
    username = session.get('username', '游客')
    
    if 'is_logged' in session:  # 检查键是否存在
        return f'当前用户: {username}'
    return '请先登录'
```
删除 Session 数据：
```python
@app.route('/logout')
def logout():
    # 删除特定键
    session.pop('user_id', None)  # 第二个参数是默认值，键不存在时返回
    
    # 清除所有数据
    session.clear()
    return '已退出登录'
```
实际应用示例：
```python
@app.route('/cart/add/<int:product_id>')
def add_to_cart(product_id):
    # 获取购物车，如果不存在则创建空列表
    cart = session.get('cart', [])
    cart.append(product_id)
    # 更新购物车
    session['cart'] = cart
    return '添加成功'

@app.route('/cart/view')
def view_cart():
    # 获取购物车内容
    cart = session.get('cart', [])
    return f'购物车内容: {cart}'
```

```python
from flask import Flask, session, redirect, url_for, escape, request

app = Flask(__name__)

# Set the secret key to some random bytes. Keep this really secret!
app.secret_key = b'_5#y2L"F4Q8z\n\xec]/'

@app.route('/')
def index():
    if 'username' in session:
        return 'Logged in as %s' % escape(session['username'])
    return 'You are not logged in'

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        session['username'] = request.form['username']
        return redirect(url_for('index'))
    return '''
        <form method="post">
            <p><input type=text name=username>
            <p><input type=submit value=Login>
        </form>
    '''

@app.route('/logout')
def logout():
    # remove the username from the session if it's there
    session.pop('username', None)
    return redirect(url_for('index'))
```
#### 如何修改/输出 http header 数据 
##### 1. 输出
```python
from flask import request

@app.route('/api')
def api():
    # 获取请求头
    user_agent = request.headers.get('User-Agent')
    auth_token = request.headers.get('Authorization')
    return 'Request processed'
```
##### 2. 修改
可以通过 `response`对象来修改响应头：
```python
from flask import make_response, Response

@app.route('/api')
def api():
    # 方式1：使用 make_response
    response = make_response('Hello World')
    response.headers['Content-Type'] = 'text/plain'
    response.headers['X-Custom-Header'] = 'Custom Value'
    return response

@app.route('/download')
def download():
    # 方式2：直接返回元组 (response, status, headers)
    return 'File Content', 200, {
        'Content-Type': 'text/plain',
        'Content-Disposition': 'attachment; filename=file.txt'
    }

@app.route('/json')
def json_api():
    # 方式3：使用 Response 类
    headers = {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'GET, POST'
    }
    response = Response(
        'API Response',
        status=200,
        headers=headers
    )
    # 也可以后续添加头部
    response.headers['X-Rate-Limit'] = '100'
    return response
```
常见应用场景：
- 设置 CORS 跨域头：
```python
@app.route('/api')
def api():
    response = make_response('API Response')
    response.headers['Access-Control-Allow-Origin'] = '*'
    return response
```
- 设置缓存控制：
```python
@app.route('/static-content')
def static_content():
    response = make_response('Static Content')
    response.headers['Cache-Control'] = 'public, max-age=3600'
    return response
```
- 设置文件下载：
```python
@app.route('/download')
def download():
    response = make_response('File Content')
    response.headers['Content-Disposition'] = 'attachment; filename=report.txt'
    response.headers['Content-Type'] = 'text/plain'
    return response
```
- 全局设置响应头：
```python
@app.after_request
def after_request(response):
    response.headers['Server'] = 'MyServer'
    response.headers['X-Content-Type-Options'] = 'nosniff'
    return response
```

#### 如何部署app程序 
##### 1. 使用 Gunicorn + Nginx（推荐生产环境、中小型项目） ：
```python
# wsgi.py
from myapp import app

if __name__ == "__main__":
    app.run()
```

```nginx
# nginx 配置
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```
启动命令：
```bash
gunicorn -w 4 -b 127.0.0.1:8000 wsgi:app
```
##### 2. 使用 Docker 容器化部署 （大型项目：Docker + K8s）：
```dockerfile
# Dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["gunicorn", "-w", "4", "-b", "0.0.0.0:8000", "wsgi:app"]
```

```yaml
# docker-compose.yml
version: '3'
services:
  web:
    build: .
    ports:
      - "8000:8000"
  nginx:
    image: nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
```

#### 如何配置开发环境 
配置 python3 环境
安装基本依赖：
```python
pip install flask
pip install python-dotenv
```

#### 如何配置静态文件访问 
提供文件下载：
```python
from flask import send_from_directory

@app.route('/download/<path:filename>')
def download_file(filename):
    return send_from_directory('static/downloads',
                             filename,
                             as_attachment=True)
```

#### 如何访问数据库 
##### 1. 使用 SQLAlchemy ORM（推荐） ：
```python
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///database.db'
db = SQLAlchemy(app)

# 定义模型
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True)
    email = db.Column(db.String(120), unique=True)

# 创建表
db.create_all()

# 增删改查操作
@app.route('/add_user')
def add_user():
    user = User(username='test', email='test@example.com')
    db.session.add(user)
    db.session.commit()
    return 'User added'
```
##### 2. 使用原生 SQL：
```python
import sqlite3
from flask import g

def get_db():
    if 'db' not in g:
        g.db = sqlite3.connect('database.db')
        g.db.row_factory = sqlite3.Row
    return g.db

@app.teardown_appcontext
def close_db(error):
    db = g.pop('db', None)
    if db is not None:
        db.close()

@app.route('/users')
def get_users():
    db = get_db()
    cur = db.execute('SELECT * FROM users')
    users = cur.fetchall()
    return str(users)
```
##### 3. 使用连接池（适合高并发） ：
```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from sqlalchemy import create_engine
from sqlalchemy.pool import QueuePool

app = Flask(__name__)

# 创建连接池
engine = create_engine('mysql://user:pass@localhost/db',
                      poolclass=QueuePool,
                      pool_size=5,
                      max_overflow=10,
                      pool_timeout=30)

def get_connection():
    return engine.connect()

@app.route('/data')
def get_data():
    with get_connection() as conn:
        result = conn.execute('SELECT * FROM users')
        return str(result.fetchall())
```


#### 是否支持ORM 
支持 ORM，主要有以下几种方式：
##### 1. Flask-SQLAlchemy（最推荐） ：
- Flask 官方推荐的 ORM 扩展
- 基于 SQLAlchemy，但更易于使用
- 提供了与 Flask 的深度集成
```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql://username:password@localhost/db_name'
db = SQLAlchemy(app)

# 定义模型
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True)
    email = db.Column(db.String(120), unique=True)
```

##### 2. 原生 SQLAIchemy
- 可以直接使用 SQLAlchemy
- 更灵活，但需要更多配置
```python
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

engine = create_engine('mysql://username:password@localhost/db_name')
Base = declarative_base()
Session = sessionmaker(bind=engine)

class User(Base):
    __tablename__ = 'users'
    id = Column(Integer, primary_key=True)
    username = Column(String(80), unique=True)
```

#### 如何维护表结构的变更 
Flask 中维护表结构变更主要通过 Flask-Migrate 扩展来实现，这是基于 Alembic 的数据库迁移工具：
1.基本设置：
```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///app.db'
db = SQLAlchemy(app)
migrate = Migrate(app, db)
```
2.创建迁移脚本
```python
# 初始化迁移环境
flask db init
# 创建迁移脚本
flask db migrate -m "创建用户表"
# 应用迁移
flask db upgrade
```

#### 如何定义/组织/初始化 数据表 
##### 1. 使用 Flask-SQLAlchemy 定义模型 ：
```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///app.db'
db = SQLAlchemy(app)

# 定义基本模型
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True)
    email = db.Column(db.String(120), unique=True)

# 定义关联关系
class Post(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(100))
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'))
    user = db.relationship('User', backref='posts')
```
##### 2. 组织模型文件 ：
```python
# models/__init__.py
from flask_sqlalchemy import SQLAlchemy
db = SQLAlchemy()

# models/user.py
from . import db

class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80))

# models/post.py
from . import db

class Post(db.Model):
    __tablename__ = 'posts'
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(100))
```
##### 3. 初始化数据表：
```python
# 方式1：直接创建
def init_db():
    db.create_all()
    
    # 添加初始数据
    admin = User(username='admin', email='admin@example.com')
    db.session.add(admin)
    db.session.commit()

# 方式2：使用 Flask CLI 命令
@app.cli.command('init-db')
def init_db_command():
    db.create_all()
    click.echo('数据库已初始化')
```
##### 4. 使用 Flask-Migrate 管理数据库版本 ：
```python
from flask_migrate import Migrate

migrate = Migrate(app, db)

# 初始化迁移环境
flask db init
# 创建迁移脚本
flask db migrate -m "初始化数据表"
# 应用迁移
flask db upgrade
```
#### 如何对接orm系统和现有的表结构 
##### 1. 使用 Table 反射（自动映射） ：
```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql://user:pass@localhost/db'
db = SQLAlchemy(app)

# 自动反射现有表结构
db.Model.metadata.reflect(bind=db.engine)

# 创建模型类
class User(db.Model):
    __table__ = db.Model.metadata.tables['users']  # 映射现有表
```
##### 2. 手动定义模型并指定表名 ：
```python
class User(db.Model):
    __tablename__ = 'existing_users_table'  # 指定现有表名
    
    # 定义与现有表结构匹配的字段
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80))
    email = db.Column(db.String(120))
```
##### 3. 使用动态模型创建 ：
```python
def create_model_from_table(table_name):
    class DynamicModel(db.Model):
        __table__ = db.Model.metadata.tables[table_name]
    
    return DynamicModel

# 使用
UserModel = create_model_from_table('users')
users = UserModel.query.all()
```
##### 4. 混合使用自定义字段和现有表 ：
```python
class User(db.Model):
    __table__ = db.Model.metadata.tables['users']
    
    # 添加额外的非数据库字段
    @property
    def full_name(self):
        return f"{self.first_name} {self.last_name}"
    
    # 添加自定义方法
    def check_password(self, password):
        return check_password_hash(self.password_hash, password)
```
#### 掌握最基本的add/delete/按字段查询/count/slice/order by 
使用 Flask-SQLAIchemy 中最基本的数据库操作：
##### 1. 添加数据：
```python
# 单条添加
@app.route('/add_user')
def add_user():
    user = User(username='test', email='test@example.com')
    db.session.add(user)
    db.session.commit()
    return 'User added'

# 批量添加
@app.route('/add_users')
def add_users():
    users = [
        User(username='user1', email='user1@example.com'),
        User(username='user2', email='user2@example.com')
    ]
    db.session.add_all(users)
    db.session.commit()
    return 'Users added'
```
##### 2. 删除数据：
```python
# 按条件删除
@app.route('/delete_user/<int:user_id>')
def delete_user(user_id):
    User.query.filter_by(id=user_id).delete()
    # 或者
    user = User.query.get(user_id)
    db.session.delete(user)
    db.session.commit()
    return 'User deleted'
```
##### 3. 按字段查询：
```python
# 单条件查询
users = User.query.filter_by(username='test').all()
user = User.query.filter_by(username='test').first()

# 多条件查询
users = User.query.filter(
    User.username.like('%test%'),
    User.email.endswith('@example.com')
).all()

# OR 查询
from sqlalchemy import or_
users = User.query.filter(
    or_(User.username=='test', User.email=='test@example.com')
).all()
```
##### 4. 计数查询：
```python
# 总数
count = User.query.count()

# 条件计数
active_count = User.query.filter_by(is_active=True).count()
```
##### 5. 切片查询：
```python
# 分页
users = User.query.offset(10).limit(5).all()  # 跳过10条，取5条

# 使用 paginate
page = User.query.paginate(page=2, per_page=10)
users = page.items
```
##### 6. 排序：
```python
# 单字段排序
users = User.query.order_by(User.username).all()  # 升序
users = User.query.order_by(User.username.desc()).all()  # 降序

# 多字段排序
users = User.query.order_by(User.age.desc(), User.username.asc()).all()
```
注意事项：
- 增删改操作后需要调用 `db.session.commit()` 提交事务
- 查询操作使用 `all()` 返回列表， `first()` 返回单个对象
- 复杂查询可以使用 `filter()` 配合 SQLAlchemy 的表达式
- 建议使用 `get_or_404()` 替代 `get()` 自动处理不存在的情况
#### 如何直接使用 sql 访问数据库 不支持orm (这样的web框架，不用也罢) 
```python
import sqlite3
from flask import g

def get_db():
    if 'db' not in g:
        g.db = sqlite3.connect('database.db')
        g.db.row_factory = sqlite3.Row
    return g.db

@app.teardown_appcontext
def close_db(error):
    db = g.pop('db', None)
    if db is not None:
        db.close()

@app.route('/users')
def get_users():
    db = get_db()
    cur = db.execute('SELECT * FROM users')
    users = cur.fetchall()
    return str(users)
```
#### 如何通过http get/post 获取远程数据 
##### 1. 使用 requests 库：
```python
import requests
from flask import Flask

app = Flask(__name__)

@app.route('/fetch_data')
def fetch_data():
    # GET 请求
    response = requests.get('https://api.example.com/data')
    data = response.json()
    # POST 请求
    post_data = {
        'name': 'test',
        'age': 25
    }
    response = requests.post('https://api.example.com/users', json=post_data)
    
    return {'status': 'success', 'data': data}
```
##### 2. 使用 aiohttp 进行异步请求 ：
```python
import aiohttp
import asyncio
from flask import Flask

app = Flask(__name__)

@app.route('/async_fetch')
async def async_fetch():
    async with aiohttp.ClientSession() as session:
        # 异步 GET 请求
        async with session.get('https://api.example.com/data') as response:
            data = await response.json()
            
        # 异步 POST 请求
        post_data = {'name': 'test'}
        async with session.post('https://api.example.com/users', json=post_data) as response:
            result = await response.json()
            
    return {'data': data, 'result': result}
```
#### 如何 parse json 
##### 1. 从请求中获取 JSON 数据：
```python
from flask import request

@app.route('/api', methods=['POST'])
def handle_json():
    # 方法1：使用 get_json()
    data = request.get_json()
    # 方法2：使用 json 属性（不推荐，因为没有错误处理）
    data = request.json
    # 带参数的 get_json()
    data = request.get_json(force=True)  # 即使 Content-Type 不是 application/json 也尝试解析
    data = request.get_json(silent=True)  # 解析失败时返回 None 而不是抛出异常
    return {'message': f'收到数据: {data}'}
```
##### 2. 解析 JSON 字符串 ：
```python
from flask import json

@app.route('/parse')
def parse_json_string():
    json_str = '{"name": "张三", "age": 25}'
    # 解析 JSON 字符串
    data = json.loads(json_str)
    return f'姓名: {data["name"]}, 年龄: {data["age"]}'
```


#### 如何 parse xml 
##### 1. 使用内置的 xml.etree.ElementTree ：
```python
from flask import Flask, request
import xml.etree.ElementTree as ET

@app.route('/parse_xml', methods=['POST'])
def parse_xml():
    # 从请求中获取 XML 数据
    xml_data = request.data
    # 解析 XML
    root = ET.fromstring(xml_data)
    # 访问 XML 元素
    for child in root:
        print(child.tag, child.attrib)
    # 查找特定元素
    for user in root.findall('user'):
        name = user.find('name').text
        age = user.find('age').text
    return {'message': 'XML parsed successfully'}
```
##### 2. 使用 lxml 库（推荐，性能更好） ：
```python
from flask import Flask, request
from lxml import etree

@app.route('/parse_xml', methods=['POST'])
def parse_xml():
    # 解析 XML
    tree = etree.fromstring(request.data)
    # XPath 查询
    names = tree.xpath('//user/name/text()')
    ages = tree.xpath('//user/age/text()')
    return {
        'names': names,
        'ages': ages
    }
```
##### 3. 使用 `xmltodict` 库（转换为字典格式）：
```python
from flask import Flask, request
import xmltodict

@app.route('/parse_xml', methods=['POST'])
def parse_xml():
    # XML 转换为字典
    data_dict = xmltodict.parse(request.data)
    # 访问数据
    users = data_dict['root']['user']
    return {'data': data_dict}
```

#### 如何输出为 json 
JSON 格式的响应是常见的，用 Flask 写这样的 API 是很容易上手的。如果从视图 返回一个 `dict` ，那么它会被转换为一个 JSON 响应。
```python
@app.route("/me")
def me_api():
    user = get_current_user()
    return {
        "username": user.username,
        "theme": user.theme,
        "image": url_for("user_image", filename=user.image),
    }
```
如果 `dict` 还不能满足需求，还需要创建其他类型的 JSON 格式响应，可以使用 [`jsonify()`](https://flask.github.net.cn/api.html#flask.json.jsonify "flask.json.jsonify") 函数。该函数会序列化任何支持的 JSON 数据类型。 也可以研究研究 Flask 社区扩展，以支持更复杂的应用。
```python
@app.route("/users")
def users_api():
    users = get_all_users()
    return jsonify([user.to_json() for user in users])
```

#### 如何处理状态码:404和50x 
使用 `errorhandler()`装饰器可以定制出错页面：
```python
from flask import render_template

@app.errorhandler(404)
def page_not_found(error):
    return render_template('page_not_found.html'), 404
```
注意 [`render_template()`](https://flask.github.net.cn/api.html#flask.render_template "flask.render_template") 后面的 `404` ，这表示页面对就的出错 代码是 404 ，即页面不存在。缺省情况下 200 表示：一切正常。
#### 如何处理文件上传
用 Flask 处理文件上传很容易，只要确保不要忘记在你的 HTML 表单中设置 `enctype="multipart/form-data"` 属性就可以了。否则浏览器将不会传送你的文件。

已上传的文件被储存在内存或文件系统的临时位置。你可以通过请求对象 `files` 属性来访问上传的文件。每个上传的文件都储存在这个 字典型属性中。这个属性基本和标准 Python `file` 对象一样，另外多出一个 用于把上传文件保存到服务器的文件系统中的 [`save()`](https://werkzeug.palletsprojects.com/en/0.15.x/datastructures/#werkzeug.datastructures.FileStorage.save "(in Werkzeug v0.15.x)") 方法。
```python
from flask import request

@app.route('/upload', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['the_file']
        f.save('/var/www/uploads/uploaded_file.txt')
    ...
```
如果想要知道文件上传之前其在客户端系统中的名称，可以使用 [`filename`](https://werkzeug.palletsprojects.com/en/0.15.x/datastructures/#werkzeug.datastructures.FileStorage.filename "(in Werkzeug v0.15.x)") 属性。但是请牢记这个值是 可以伪造的，永远不要信任这个值。如果想要把客户端的文件名作为服务器上的文件名， 可以通过 Werkzeug 提供的 [`secure_filename()`](https://werkzeug.palletsprojects.com/en/0.15.x/utils/#werkzeug.utils.secure_filename "(in Werkzeug v0.15.x)") 函数:
```python
from flask import request
from werkzeug.utils import secure_filename

@app.route('/upload', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['the_file']
        f.save('/var/www/uploads/' + secure_filename(f.filename))
    ...
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