## 前言：gorm 痛点
gorm 查询语句：
```go
var user models.UserModel 
global.DB.Take(&user, "username = ?", "fengfeng")
```
通常在写 gorm 查询语句时，字段名是写死在代码中的，如果字段名发生改变，程序只会在运行时报错。但这种字段发生改变的错误最好是在程序编译的时候暴露，不然只有在程序运行时才能暴露出来。并且 gorm 的 `create`，`where` 等方法，参数接收的都是`any`类型，但是如果错传类型，也是要在运行时才能知道错误。
使用 gen 可以很好地解决上述问题。

## 一、通过表生成模型
如果是现有表，再写代码，可以使用这种方法

先模拟建个表
```sql
-- 创建用户表 
CREATE TABLE users ( 
	id INT AUTO_INCREMENT PRIMARY KEY COMMENT '用户ID', 
	name VARCHAR(50) NOT NULL COMMENT '用户名', 
	age INT NOT NULL COMMENT '年龄', 
	addr VARCHAR(100) NOT NULL COMMENT '地址', 
	createdAt DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间' 
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户信息表'; 

-- 创建视频表 
CREATE TABLE videos( 
	id INT AUTO_INCREMENT PRIMARY KEY COMMENT '视频ID', 
	user_id INT NOT NULL COMMENT '用户ID（关联用户表）', 
	title VARCHAR(100) NOT NULL COMMENT '视频标题', 
	src VARCHAR(255) NOT NULL COMMENT '视频地址', 
	createdAt DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间', 
	-- 添加外键约束 
	FOREIGN KEY (user_id) REFERENCES users (id) 
		ON DELETE CASCADE 
		ON UPDATE CASCADE 
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='视频信息表';
```

通过`GenerateAllTable`生成gorm模型，再通过`ApplyBasic`生成代码
```go
package main

import (
	"fmt"
	"gen_study/models"
	"gorm.io/driver/mysql"
	"gorm.io/gen"
	"gorm.io/gorm"
)

func main() {
	// 初始化代码生成器
	g := gen.NewGenerator(gen.Config{
		OutPath:       "./query",         // 生成的代码输出目录
		ModelPkgPath:  "models",          // 模型代码包路径
		Mode:          gen.WithDefaultQuery | gen.WithoutContext, // 启用默认查询和链式接口
		FieldNullable: true,              // 允许 NULL 的字段生成指针类型
	})
	// 配置数据库连接
	dsn := "root:root@tcp(127.0.0.1:3306)/gen_study?charset=utf8mb4&parseTime=True&loc=Local"
	db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
	if err != nil {
		fmt.Println("数据库连接失败:", err)
		return
	}
	// 使用数据库并生成代码
	g.UseDB(db)
	g.GenerateAllTable() // 根据所有数据表生成模型
	g.Execute()          // 执行代码生成
}

```

`GenerateAllTable`是生成这个库下所有表的

也可以指定表生成模型代码
```go
g.GenerateModel("users")
```

> 这种方式生成的，是不带外键信息的
### 生成外键模型
生成带外键信息的模型稍微复杂一些
```go
// 生成视频模型，并关联用户（BelongsTo）
videoModel := g.GenerateModel("videos",
	gen.FieldRelate(
		field.BelongsTo,
		"User",
		g.GenerateModel("users"),
		&field.RelateConfig{
			GORMTag: field.GormTag{
				"foreignKey": []string{"UserID"},
			},
		},
	),
)

// 生成用户模型，并关联多个视频（HasMany）
g.GenerateModel("users",
	gen.FieldRelate(
		field.HasMany,
		"Videos",
		videoModel,
		&field.RelateConfig{
			GORMTag: field.GormTag{
				"foreignKey": []string{"UserID"},
			},
		},
	),
)

```

## 二、通过模型生成 gen 代码
有了模型之后，就可以通过模型生成gen代码
```go
g.ApplyBasic(models.User{}, models.Video{})
```
## 三、增删查改操作
有了生成的gen代码

对于增删改查操作会非常简单
### 创建记录
创建单条
```go
// 设置默认数据库连接
query.SetDefault(db)

// 创建一个用户对象
user := models.User{
	Name: "枫枫",
	Age:  21,
	Addr: "cs",
}

// 执行插入操作
err := query.User.Create(&user)

// 输出结果
fmt.Println(user, err)

```
创建多条
```go
// 创建多个用户对象
u1 := models.User{
	Name: "枫枫",
	Age:  21,
	Addr: "cs",
}

u2 := models.User{
	Name: "zhangsan",
	Age:  22,
	Addr: "cs",
}

// 将用户对象转换为指针列表
var userList = []*models.User{&u1, &u2}

// 执行批量插入操作
err := query.User.Create(userList...)

// 输出结果（包括插入后生成的 ID 和错误信息）
fmt.Println(u1, u2, err)
```
### 查询语句
```go
// 🔍 查全部用户
users, _ := query.User.Find()
for _, user := range users {
	fmt.Println(user)
}

// 🔍 条件查询：查找名称为 "zhangsan" 的用户
user, _ := query.User.
	Where(query.User.Name.Eq("zhangsan")).
	Take()
fmt.Println(user)

// 🔍 外键查询：查找 ID 为 1 的用户所关联的视频列表
videos, _ := query.User.Videos.
	Model(&models.User{ID: 1}).
	Find()
for _, video := range videos {
	fmt.Println(video)
}
```
### 更新
```go
// 🔧 更新单列（必须带 WHERE 条件）
res, err := query.User.
	Where(query.User.ID.Eq(1)).
	Update(query.User.Name, "fengfeng")
fmt.Println(res, err)

// 🔧 更新多列（只能更新非零值字段，即不会更新零值如 ""、0、false）
res, err = query.User.
	Where(query.User.ID.Gte(1)).
	Updates(models.User{
		Name: "zhangsan", // 只更新非零字段
	})
fmt.Println(res, err)
```
### 删除
```go
// 根据主键删除 
res, err := query.User.Where(query.User.ID.Eq(1)).Delete() 
fmt.Println(res, err) 
// 根据model删除 
res, err = query.User.Delete(&models.User{ID: 2}) 
fmt.Println(res, err)
```

