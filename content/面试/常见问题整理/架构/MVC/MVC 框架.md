MVC 是一种软件设计模式

| 缩写  | 全称             | 含义                   |
| --- | -------------- | -------------------- |
| M   | **Model**      | 模型层，处理数据和业务逻辑        |
| V   | **View**       | 视图层，负责 UI 展示         |
| C   | **Controller** | 控制器层，处理用户输入并协调 M 和 V |
> MVC 模式的主要目的是 **解耦** —— 将表示层、业务逻辑层和输入控制分离，各层职责清晰，便于开发、测试和维护。

**Model（模型层）**
- 负责：**数据的存取、业务逻辑处理**
- 和数据库打交道，如：增删改查
- 在 Web 框架中通常是 ORM（对象关系映射）操作
```go
type User struct {
    ID    int
    Name  string
    Email string
}
```
**View（视图层）**
- 负责：**界面展示**，即用户看到的内容
- 通常是 HTML 页面、模板引擎、前端页面等
- 不包含任何业务逻辑

**Controller（控制器层）**
- 负责：**接收请求、调用模型、选择视图**
- 是用户和系统之间的桥梁
- 处理请求参数、调用逻辑、返回结果
```go
func GetUserHandler(c *gin.Context) {
    id := c.Param("id")
    user := model.GetUserByID(id)
    c.HTML(http.StatusOK, "user.html", gin.H{"User": user})
}
```

##### 作用
- 结构清晰，职责分离  
- 易于测试和维护  
- 支持多人协作开发（前端/后端分工）  
- 提高代码复用性