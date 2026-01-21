## 一、什么是 Swagger
​ Swagger 本质上是一种用于描述使用`JSON`表示的`RESTful API`的接口描述语言。Swagger 与一组开源软件工具一起使用，以设计、构建、记录和使用`RESTful Web`服务。Swagger 包括自动文档，代码生成和测试用例生成。
## 二、安装 Swagger
在项目的根目录下执行以下命令：
```bash
go get -u github.com/swaggo/swag/cmd/swag # Go1.17以前使用该命令 
go install github.com/swaggo/swag/cmd/swag@latest # Go1.17及之后的版本使用该命令 
go get -u github.com/swaggo/gin-swagger 
go get -u github.com/swaggo/files
```
检验是否安装成功：
```bash
swag -v
swag version v1.16.4 # 出现 swag 版本号即说明安装成功
```


## 三、使用 gin-swagger 自动生成接口文档
使用 `gin-swagger` 为项目自动生成接口文档，需要下列步骤：
#### 1. 写入注解：根据 swagger 规范为项目的 `API` 接口编写注解
##### （1）为项目添加注解
在程序入口 `main` 方法写入项目相关的信息：
```go
// @title         待办事项 API 文档  
// @version    1.0  
// @description 这是详细介绍待办事项的 API 文档  
//  
// @contact.name    huang  
// @contact.email   nanguatou10@gmail
func main() {  
	// ……
    r.Run(":9090")  
}
```
##### （2）在接口方法写入注解
```go
// CreateTodo 创建一个新的待办事项  
//  @Summary      创建待办事项  
//  @Description   接收前端传来的 JSON，创建一个 Todo 项目  
//  @Tags        Todo  
//  @Accept          json  
//  @Produce      json  
//  @Param       todo   body      models.Todo             true   "待办事项内容"  
//  @Success      200       {object}   models.TodoResponse       "创建成功返回的结构体"  
//  @Failure      400       {object}   models.ErrorResponse   "请求参数错误"  
//  @Router          /todo [post]  
func CreateTodo(c *gin.Context) {}
```

##### 注解说明：

|    注解    |                         描述                         |
| :------: | :------------------------------------------------: |
| @Summary |                      对该接口的描述                       |
| @Accept  |                    该接口接收请求的编码类型                    |
| @Produce |                     该接口返回的数据类型                     |
|  @Param  | 表示参数，分别为：参数名称、参数类型、数据类型、是否必填、注解、属性(可选参数),参数之间用空格隔开 |
| @Success |       表示请求成功后返回，它有以下参数： 请求返回状态码、参数类型、数据类型、注释       |
| @Failure |       表示请求失败后返回，它有以下参数：请求返回状态码、参数类型、数据类型、注解        |
| @Router  |               路由，从左至右分别为：路由地址和HTTP方法               |
其中`Param` 的参数类型有以下几种：
- query 形如 `/articles?title=xxxxx&status=xxxx`
- body 需要将数据放到 body 中进行请求
- path 形如 `/articles/1`

上述注解中，使用到了`@Success 200 {object} models.TodoResponse "创建成功返回的结构体"`,我们专门定义一个针对Swagger的对象，用来Swagger接口文档的展示。在`models/todo.go`文件中，定义一个model：
```go
type TodoResponse struct {  
    Status int    `json:"status"`  
    Msg    string `json:"msg"`  
    Data   Todo   `json:"data"`  
}  
  
type ErrorResponse struct {  
    Error string `json:"error"`  
}
```

#### 2. 使用`swag init`命令生成`API`接口文档所需的文件
在根目录下，执行命令`swag init`，执行完成后会在根目录下新建一个 `docs` 文件夹，在该文件夹中生成`docs.go`,`swagger.json` ,`swagger.yaml`三个文件。
#### 3. 项目中集成 swagger
在项目代码中注册路由的地方按如下方式引入`gin-swagger`相关内容：
`routers.go`:
```go
import (  
    "bubble/controller"  
  
    "github.com/gin-gonic/gin"    
    "github.com/swaggo/files"    
    "github.com/swaggo/gin-swagger")
```
在`main.go`中导入`swag init`生成的 docs:
```go
import (  
    "bubble/dao"  
    _ "bubble/docs" // 导入上一步生成的docs
    "bubble/routers")
```
注册 `swagger api` 相关路由
```go
r.GET("swagger/*any", ginSwagger.WrapHandler(swaggerFiles.Handler))
```
## 四、查看接口文档
​ 重启服务端，在浏览器中访问Swagger的地址`http://127.0.0.1:8000/swagger/index.html`，即可看到如下图所示的Swagger文档展示，其中主要分为三部分。分别是项目主题信息、接口路由信息和模型信息。
![[Pasted image 20250518183314.png]]



