#### 如何定义 url router 
项目结构示意：
```bash
myproject/
├── myproject/          # 项目目录（包含settings）
│   └── urls.py         # 项目主路由
├── myapp/              # 应用目录
│   ├── views.py        # API 视图
│   ├── urls.py         # 应用路由
│   └── serializers.py  # DRF 序列化器
└── manage.py
```

##### 1. 在应用的`urls.py` 中定义路由
```python
# app_name/urls.py

from django.urls import path
from . import views  # 导入视图模块

urlpatterns = [
    path('', views.index, name='index'),  # 根路径
    path('about/', views.about, name='about'),  # about 页面
    path('article/<int:id>/', views.article_detail, name='article_detail'),  # 动态路由
]
```
##### 2. 项目的主`urls.py`中包含应用路由
```python
# project_name/urls.py

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_name.urls')),  # 引入应用的 urls.py
]
```

> path() 方法介绍：
> - `route`（路径）：要匹配的 URL 片段
> - `view`（视图）：处理这个 URL 请求的视图函数或类视图
> - `kwargs`（可选参数）：可以向视图函数手动传递额外参数（不常用）
> - `name`（路由命名）：给 URL 取名字，方便用 `reverse()` 或模板中的 `{% url 'name' %}` 反查

#### 如何组织 request handler 函数 
在 Django 中，**Request Handler**（请求处理函数）通常是指处理请求的**视图函数（View）**，它是 Web 应用响应用户请求的核心。Django 提供两种主要方式来组织 request handler：
##### 1. 函数式视图
示例：
```python
# app/views.py

from django.http import HttpResponse

def my_view(request):
    return HttpResponse("Hello, world!")
```
在 `urls.py`中路由绑定：
```python
# app/urls.py

from django.urls import path
from . import views

urlpatterns = [
    path('hello/', views.my_view),
]
```
##### 2. 类视图
Django 也支持用类来组织请求处理逻辑，更加模块化、易于继承与扩展：
```python
# app/views.py

from django.http import HttpResponse
from django.views import View

class HelloView(View):
    def get(self, request):
        return HttpResponse("Hello from class view!")
```
路由绑定：
```python
# app/urls.py

from django.urls import path
from .views import HelloView

urlpatterns = [
    path('hello/', HelloView.as_view()),
]
```
#### 写一个最简单的request handler 函数 【见上一个问题】
#### 如何从get/post请求中取出参数 
##### 1. URL 路径参数
```python
# urls.py
path('user/<int:user_id>/', views.user_detail)

# views.py
def user_detail(request, user_id):
    return HttpResponse(f"用户ID是 {user_id}")

```
##### 2. 查询参数
比如 URL：`/search/?q=django&page=2`
```python
def search(request):
    query = request.GET.get('q', '')
    page = request.GET.get('page', '1')
    return HttpResponse(f"搜索关键词：{query}, 页码：{page}")

```

表单或普通 POST 请求通过 `request.POST` 获取：
```python
def submit_form(request):
    if request.method == 'POST':
        name = request.POST.get('name', '')
        email = request.POST.get('email', '')
        return HttpResponse(f"姓名：{name}, 邮箱：{email}")

```

##### 3. 请求体参数
```python
import json

def api_view(request):
    if request.method == 'POST':
        data = json.loads(request.body)
        name = data.get('name')
        return JsonResponse({'message': f'Hello, {name}'})

```

#### 如何定义全局url 拦截函数 
##### 1. 定义全局 URL 拦截中间件
```python
# myapp/middleware.py

from django.http import HttpResponseRedirect
from django.urls import reverse

class AuthRequiredMiddleware:
    """
    拦截未登录用户访问敏感路径（如 /dashboard/）
    """
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        # 需要拦截的前缀路径
        protected_paths = ['/dashboard/', '/admin-area/']

        # 如果路径命中并且用户未登录，则重定向
        if any(request.path.startswith(p) for p in protected_paths):
            if not request.user.is_authenticated:
                return HttpResponseRedirect(reverse('login'))

        # 放行
        return self.get_response(request)
```

##### 2. 在 `settings.py` 注册中间件
```python
MIDDLEWARE = [
    # ... 其他中间件
    'myapp.middleware.AuthRequiredMiddleware',  # 👈 添加在适当位置
]
```
> 注意中间件顺序可能影响行为，通常放在认证中间件后面更稳妥。

#### 如何获取/修改/存储 cookie,session数据 
#### 如何修改/输出 http header 数据 
#### 如何部署app程序 
#### 如何配置开发环境 
#### 如何配置静态文件访问 
#### 如何访问数据库 
#### 是否支持ORM 
#### 如何维护表结构的变更 
#### 如何定义/组织/初始化 数据表 
#### 如何对接orm系统和现有的表结构 
#### 掌握最基本的add/delete/按字段查询/count/slice/order by 
#### 如何直接使用sql 访问数据库 不支持orm (这样的web框架，不用也罢) 
#### 【如何使用模板系统 
#### 如何组织/访问 模板文件的目录结构 
#### 如何在模板中嵌入代码 
#### 模板是否支持继承结构 
#### 模板之间如何include 
#### 如何自定义模板函数 】
#### 如何通过http get/post 获取远程数据 
#### 如何parse json 
#### 如何parse xml 
#### 如何输出为 json 
#### 如何处理状态码:404和50x 
#### 如何处理文件上传