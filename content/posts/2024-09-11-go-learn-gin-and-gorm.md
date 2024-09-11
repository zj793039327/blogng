---
layout: post
title: Go Microservices Using Gin And GORM（翻译）
date: 2024-09-10 10:00:00
author: jojoster
tags:
  - go
categories: tech
---

原始文章：
https://medium.com/@prithuadhikary/go-microservices-using-gin-and-gorm-72938b3b56b4
此处是译文，主要针对初学者，没有接触过go的同学，手把手的教学用go写微服务。
<!--more-->

![希望我是一杯伏特加😋](http://img.skydrift.cn/1725603868.png?imageMogr2/thumbnail/!70p)
## 简洁的web框架：Gin

Gin是一个多功能的web框架，使用go语言开发。框架对HTTP请求和响应做了良好的封装，提供简单可用的函数，可以将表单、Json以及各类参数转化为数据结构（struct），并且还提供了参数的校验功能(**validation**) 。这些函数可以轻松的将接口渲染成 JSON 或者 XML返回，或者是渲染为模板驱动的HTML响应。

Gin还提供了编写中间件的方案（**middlewares**），举例来说是一些涉及到切面的问题，比如使用JWT进行身份校验、`population of the request context with parsed claims`[^fn1]、按照当今分布式trace工具的要求，在header中增加span等

[^fn1]: 没想好这段怎么翻译。容我仔细想想。

本篇文章中，我们会讲到这些问题
1. 使用Gin创建一个微服务
2. 在微服务中，使用GORM（Golang Object Relational Mapping）针对Postgres数据库进行开发
## User 微服务

接下来的演示案例，是编写一个用户登录服务，提供REST风格的接口。功能如下：
1. 接口会接收一个完整的JSON结构，通过http的payload发送
2. 并且进行校验（validate）
3. 使用GORM结构转化，并且实现持久化。数据库则选用postgres。
4. 进行异常处理（error handling）
5. 进行服务的身份验证（authentication）。
## 初始化服务引入Gin和GORM

一行代码足矣
```bash
mkdir user-service && \  
cd user-service && \  
go mod init github.com/prithuadhikary/user-service
```

命令会在 `user-service` 这个文件夹中，创建和初始化服务所需的代码，在这个文件夹下，需要有一个 `go.mod` 文件。

执行下面的命令，引入框架
```bash
go get -u github.com/gin-gonic/gin
go get -u gorm.io/gorm
```

为了连接数据库，需要postgres的连接器

```bash
go get -u gorm.io/driver/postgres
```
依赖已经都搞定了
## 初始化GORM

在编写持久层之前，需要初始化GROM。包括下面的步骤

1. 定义对象模型（domain classes），并且需要通过属性和tag让GROM知道数据库结构（schema）
2. 连接数据库
3. 自动同步结构以及对象（automigration）举例，让GORM创建需要的表、主外键关系等。
## 分层架构

传统的企业应用中，分层设计是一个通用的最佳实践，即，将系统逻辑打散，编排到三个维护性较好的 “层（The Layer）” 中，这三层分别是
* **Controller**，控制层，承担request（大部分是HTTP）数据的接收，解析请求体，参数，Header，Path等参数。创建service层的对象，调用实现功能，得到结果后，将结果写入到response中去。
* **Service**，服务层，主要封装业务逻辑，和Repository交互。
* **Repository**，持久层，关注细节，和domain对象打交道，执行数据库保存等。

接下来开始逐步构建，从最底部的Repository层开始。

## 定义User 领域（domain）—— User Struct Type

Repository层一般操作 `domain` 的实体。这些实体其实是数据库的表结构在服务中的映射。
示例从简，创建一个 `domain`文件夹，编写`user.go`
```go
package domain

import "gorm.io/gorm"

type User struct {
   gorm.Model
   ID       uuid.UUID `gorm:"type:uuid;primarykey"`
   Username string
   Password string
   Role     string
}
```

上面的domain中，有4个属性，没有疑问，还有一个 `gorm.Model`。这里需要额外解释下：

由于go中没有继承（inheritance）机制，需要用组合（composition）实现多态，下面讲一下如何用组合实现类似继承的能力。
比如有两个类型如下：
```go
type Vodka struct {
   AlcoholPercent float32
}

func (vodka Vodka) Hangover()  {
   fmt.Println("Tasteless Hangover!")
}

type BloodyMary struct {
   Vodka
   Ingredients []string
}
```
接下来定一个`BloodyMary`的实例，
```go
var myPeg BloodyMary
```
就可以调用Vodka的方法了
```go
myPeg.AlcoholPercent
```
go语言中，这个功能叫类型嵌入（**Type Embedding**），其实，也可以这么调用
```go
myPeg.Vodka.AlcoholPercent
```

相当于是将 `Vodka` 嵌入到了 `BloodMary`中👻。之所以可以直接调用，是因为go语言中，对于嵌入的类型，会将其属性和方法都提升（promoted）到主类型中。`myPeg.Hangover()`也可以直接调用。
但是，属性的提升仅限于不覆盖（shadowed）的场景。

如果主类型中也有同名的方法，则会覆盖嵌入类型中的。比如在`BloodMary`中也定定义了`Hangover` 方法如下：
```go
func (bloodyMary BloodyMary) Hangover()  {
   fmt.Println("Hangover yet tasty!")
}
```
调用类型中的同名方法，会覆盖嵌入类型中的，优先级比较高。

```go
var myPeg BloodyMary
myPeg.Hangover()
// output：Hangover yet tasty!

myPeg.Vodka.Hangover()
// output: Tasteless Hangover!
```

切回正题，现在，对于类中的 `gorm.Model` 应该就不会陌生了，这个`Model`中包含了一些通常的属性，如下所示
```go
// Model a basic GoLang struct which includes the following fields: // ID, CreatedAt, UpdatedAt, DeletedAt
// It may be embedded into your model or you may build your own 
// model without it
//    type User struct {
//      gorm.Model
//    }
type Model struct {
   ID        uint `gorm:"primarykey"`
   CreatedAt time.Time
   UpdatedAt time.Time
   DeletedAt DeletedAt `gorm:"index"`
}
```

在上面的模型中，已经有了`ID`。在`User` 结构中，你可能会注意到，又定义了`ID`属性😉。
```go
package domain

import "gorm.io/gorm"

type User struct {
   gorm.Model
   ID       uuid.UUID `gorm:"type:uuid;primarykey"`
   Username string
   Password string
   Role     string
}
```
这个定义，目标就是将 `gorm.Model` 中的数字类型 `ID` 进行覆盖，用`UUID`进行替代。数字用的主键，通常满足某种自增序列，容易被钻空子，比如你可以算出来下一个。`UUID`至少可以在一张表中保证唯一。

> 重要提示：
> 假如你的程序在全世界各处运行，但是，你依然在单表中，遇到了UUID碰撞冲突，那么只有两种可能
> 1. 上帝让你下地狱
> 2. Morpheus找到了你，准备让你离开 Matrix。你一定立刻马上选择红药片🤪

适才相戏耳。在HTTP场景中，最好使用UUID，因为随机性更好，安全性也更好。尤其是你准备把主键放到HTTPGet方法的Path上去的时候。
如果你用了数字主键，那么我拿出这个请求，阁下又该如何应对。
```http
DELETE /credit-card/123
```

我相信自己说明白了，继续向下，来到初始化DB链接的环节。

## 连接数据库

这部分代码比较简单，也比较符合语言习惯。下面是主要函数，然后返回了一个用来和DB交互的指针`gorm.DB`
[代码链接](https://gist.githubusercontent.com/prithuadhikary/c06ff515db27f1315e822e5d5c640f72/raw/82e457de86b29c3a99e827f2f24fcf603dcef0d4/intialize.db.go)
```go
func InitialiseDB(dbConfig *DbConfig) (*gorm.DB, error) {
   dsn := fmt.Sprintf("host=%v user=%v password=%v dbname=%v port=%v sslmode=require TimeZone=Asia/Kolkata", dbConfig.Host, dbConfig.User, dbConfig.Password, dbConfig.DbName, dbConfig.Port)
   newLogger := logger.New(
      log.New(os.Stdout, "\r\n", log.LstdFlags), // io writer
      logger.Config{
         SlowThreshold:             time.Second, // Slow SQL threshold
         LogLevel:                  logger.Info, // Log level
         IgnoreRecordNotFoundError: true,        // Ignore ErrRecordNotFound error for logger
         Colorful:                  true,        // Disable color
      },
   )
   db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{
      Logger: newLogger,
      NamingStrategy: schema.NamingStrategy{
         TablePrefix:   dbConfig.Schema + ".",
         SingularTable: false,
      },
   })
   if err != nil {
      return nil, err
   }
   return db, err
}
type DbConfig struct {
   User     string `mapstructure:"user"`
   Password string `mapstructure:"password"`
   DbName   string `mapstructure:"dbName"`
   Host     string `mapstructure:"host"`
   Port     string `mapstructure:"port"`
   Schema   string `mapstructure:"schema"`
}
```
优秀的代码本身就是最好的注释。但是我还是解释一下
1. dsn 是链接数据库的描述信息串，包含数据的连接四要素
2. 将dsn传递给gorm的driver，`postgres.Open(dsn)`，可以拿到 gorm的 Dialector（方言）指针
3. 通过`*gorm.Config` 然后通过GORM提供的接口，可以拿到 `*gorm.DB`，大功告成。

注意，`mapstructure` 这个tag，是因为我用了viper去加载了属性，这个阶段可以不用关注，以免带来不必要的干扰。
最后，开启了GORM的 log，可以打印日志，通过`NamingStrategy` 确认开启的表结构。

可以在main方法中初始化DB，如下
```go
db, err := **InitialiseDB**(&DbConfig{  
   User:     "postgres",  
   Password: "password",  
   DbName:   "groot",  
   Host:     "localhost",  
   Port:     "5432",  
   Schema:   "users",  
})  
if err != nil {  
   panic(err)  
}err = **db.AutoMigrate(&domain.User{})**  
if err != nil {  
   panic(err)  
}
```

`db.AutoMigrate(&domain.User{})` 这句会让GORM 分析当前表结构，并且判定是否符合当前表定义，不符合的话，会根据当前代码结构自动变更db结构，失败则会抛出异常，进而引发panic。
还有一点值得注意，这个migrate尽量不会引起数据的丢失，不会执行任何**drop**命令，当前表已经不用的列，也不会删除。

现在可以用gorm.DB指针了，就是代码中的 `db`变量。现在可以创建持久层了。

## The UserRepository 持久层

首先需要定一个接口，对`User`进行持久化操作。在`repository` 文件夹下，创建一个go文件，`user.repository.go`
```go
package repository  
  
import "github.com/prithuadhikary/user-service/domain"  
  
type UserRepository interface {  
	Save(user *domain.User)  
}
```

简约而不简单。这是一个接口，只定义行为，`Save`是动作。其中首字母的大写，在go中代表是public的。大写开头的属性、方法，在package以外也可以访问到。

接口独木不成林，需要定义对应的实现。在文件中新增如下：
```go
type userRepository struct {  
	db *gorm.DB  
}
```

这个struct并没有实现接口，需要增加下面的定义才行

```go
func (repository *userRepository) Save(user *domain.User) {  
	// Implement the logic to save here.  
}
```
在 go 中，实现和接口只是口头协议（契约）的关系。接口的实现并不需要显示的用 `implement` 进行关联或者用语法定义。`UserRepository` 是接口，`userRepository` 实现，在实际的引用中，两个变量可以等同。如下
```go
func NewUserRepository(db *gorm.DB) UserRepository {
   var repository UserRepository
   repository = &userRepository{
      db: db,
   }
   return repository
}
```

这是我们定义的第一个 构造（constructor）方法。由于用了小写字母`u`定义了实现，不对外暴露，因此，为了在包外访问这个实现，因此需要构造方法允许外部访问。构造方法会返回`userRepository`的指针，让调用方作为 `UserRepository`访问。

更进一步以前，需要讨论一下方法集（method sets）。与指针不同，按照下面定义，可以获得不带指针的对象
```go
repository = userRepository{
      db: db,
}
```
但是这样是不能编译的，原因在于，结构并没有实现接口，而指针才能实现接口。
### 探讨Go语言中的接口设计Receivers And Interfaces Under The Hood — Skippable
这部分可以跳过。不会影响对于持久层的理解。

回顾一下`save`方法
```go
func (repository *userRepository) Save(user *domain.User) {
   // Implement the logic to save here.
}
```

`*userRepoitory` 称为接收器（receiver）。接收器用来承担外部的调用。可以将其理解为是方法的第一个参数。receiver 可以有两种，类型本身和类型的引用（前者不带*）。下面的例子就是直接将类型本身作为receiver。
```go
func (repository userRepository) Save(user *domain.User) {
   // Implement the logic to save here.
}
```
至此，`repository` 在方法内部是可见的，相当于第一个入参。
传递type引用还是传递type本身，都可以。`Save`方法中，value receiver 是一个全新的副本，pointer receiver则是将自己完全传递。对于后者，任何改动都会在外部生效。对于前者，所有属性，包括指针都会赋值。
```go
func printPointerAddress(aPtr *int32) {
        println("Address of pointer aPtr", &aPtr)
}
And then in main, 
a := int32(10)
ptr := &a
println("Addres of pointer ptr", &ptr)
printPointerAddress(ptr)
```
输出如下
```go
Addres of pointer ptr 0xc000084db8
Address of pointer aPtr 0xc000084dd0
```

`ptr` 和 `aPtr` 。`ptr`是一个指针，同时也是一个变量。当通过`printPointerAddress` 传递指针的时候，这个指针会被复制为`aPtr`，注意，`aPtr` 也是一个指针，指向上一个变量`ptr`。aPtr则是我们获得的，一个指向原始内存地址的句柄（handler），对于这个handler可以进行写操作、取消引用的操作。
但是你需要知道，这绝不是真正的按引用传递。😁 Because even a pointer to pointer to pointer will also get copied. It is by the virtue of the nature of pointers, it is a pass by reference.

回到定义
```go
func (repository userRepository) Save(user *domain.User) {
   // Implement the logic to save here.
}
```
还有一个隐含的问题，这两个定义在代码上都是合法的，可以编译通过，但是含义不同。
```go
var repository UserRepository

//This is implied.
repository = userRepository{
   db: db,
}

//This is also valid.
repository = &userRepository{
   db: db,
}
```

如何选取，一些经验可以听一下
* pointer reveiver：这个pointer只能赋值给接口的变量
* value reveiver：可以同时赋值给接口的变量、实现类的变量

可以简化记忆一下：接口变量存储的是reveiver，如果receiver是一个指针，那么接口存储的一定是指针receiver。如果receiver是一个value receiver的话，接口可以像是存储地址一样存储自身，这就是方法集（method sets）的含义。
如果一个变量的类型是interface，可以同时存储指针类型的赋值，或者值类型的赋值，区别如下：
1. 指针类型赋值：只包含接口的方法
2. 值类型赋值：包含接口和实现的方法。

方法集：在Go中，方法集定义了一个类型必须实现的所有方法，以满足某个接口的要求。
- 如果类型 `T` 的方法使用的是值接收者，那么这些方法属于 `T` 和 `*T` 的方法集。
- 但如果方法使用的是指针接收者，那么这些方法只属于 `*T` 的方法集，而不属于 `T`


给定一个指针，可以明确知道指针指向的value，如果拿到的是一个值，其实不能明确的推断出来地址。举例，在go中，可以由用户主动的将基础类型，重定义为用户自定义类型。如下：

 
```go
type Number interface {
   Increment() Number
}

type number int64

func (num *number) Increment() Number {
   *num++
   return num
}
n := number(5)
// The above declaration does three things:
// 1. Allocate memory
// 2. Give it the name 'n'
// 3. Assign the value number(5) to it.
// 4. Since there exists a named memory location,
//    we can take its address by &n (address of n).

nptr := &number(6)
// On the other hand, &number(6) isn't feasible
// even though number(6) could have caused an 
// allocation,
// we haven't got a named handle to its location.
// Likewise, &6 is invalid.
```

在底层，Go语言的接口是通过一个由一个类似pair的数据结构实现的。
* **itab**：第一个字 `itab` 存储了一个指向 `Itab` 结构体（源自C语言世界）的指针，该结构体包含了关于类型的元数据，接着是函数指针（如 `fun[0]`, `fun[1]` 等），这些指针指向实现该接口类型的具体类型中的相应函数。
* **data**：第二个字 `data` 存储了一个指向实现类型所持有的数据的指针。当我们将一个具体类型赋值给接口变量时，底层的数据结构，即 `data` 和 `itab` 被填充。而当我们通过接口变量（例如 `iface`）调用一个方法时，Go会执行类似以下操作的步骤。

```go
Depending upon the kind of receiver fun[0] has,
iface.itab->fun[0](iface.data,...additional parameters) 
or 
iface.itab->fun[0](*iface.data, ...additional parameters)
```

记住，`fun[0]` 的含义是 `(*(fun+0))`
问题在于，go语言有时候无法推断出来 指向 `itab` 结构的函数指针是什么，因为没有可以开始的地址。
举例来说，`&number(5)` 这个表达式，是一种类似Consts和Literal的东西。现代的C语言允许使用复合变量，因此可以定义类似这样子的实例，并且拿到地址。go也可以拿到类似这样字的结构体实例的地址，但是这一套东西，明显和常量和基础类型不一样

深入探讨到止为止，接下来继续看微服务的实现。

## 实现Save方法
很简单，`gorm.DB` 提供了一个方法`Save(interface{})`，保存数据，如下

```go
func (repository userRepository) Save(user *domain.User) {
   repository.db.Save(user)
}
```
但是有一个事情很重要，如果我们创建了一个 `domain.User`，ID字段(定义为16\[byte\])的属性就会拿到`uuid.Nil`（16\[byte\]的空值）。User中的ID需要一个UUID的值，如果没有设置，需要初始化并且设进去。
GORM Hooks可以用来解决这个问题。GORM可以针对 domain 的对象定义一些hook方法，在某些CRUD操作前后执行。当前，需要在User的create方法之前引入一个hook方法，参考GORM的文档
```go
// begin transaction  
BeforeSave  
BeforeCreate //********************************这里正合适
// save before associations  
// insert into database  
// save after associations  
AfterCreate  
AfterSave  
// commit or rollback transaction
```
完整文档请移步 [GORM Hooks 文档](https://gorm.io/docs/hooks.html)

UUID的话，需要安装 Google的 uuid包
```bash
go get -u github.com/google/uuid
```

然后定义一个`BeforeCreate` 方法，如下：
```go
func (u *User) BeforeCreate(tx *gorm.DB) (err error) {
  u.ID = uuid.New()

  return
}
```

还缺一些东西，可以顺便把 `Password` 字段一起搞定。在DB中存储 password 字段，最好是需要hash（当然加盐hash最好）。在本例中，可以将一个hex使用 `sha256` 算法进行计算散列值（此处省略密盐）

```go
func (user *User) BeforeCreate(tx *gorm.DB) error {
   user.ID = uuid.New()
   
   // Digest and store the hex representation.
   digest := sha256.Sum256([]byte(user.Password))
   user.Password = hex.EncodeToString(digest[:])

   return nil
}
```

现在开始编写service层，主要的内容是将输入转化为 `domain.User`，然后调用持久层。

## The User Service  服务层

现在开始定一些基础的struct，作为`Signup`方法的依赖。需要建立 `model`文件夹，然后创建`signup-request.go`的文件。
```go
package model

type SignupRequest struct {
   Username             string `json:"username"`
   Password             string `json:"password"`
   PasswordConfirmation string `json:"passwordConfirmation"`
}
```

其中的jsontags定义了，针对json转化的字段名，Gin也提供了其他的tag，可以针对POST，GET请求以及进行字段的校验（validation）。也支持自定义 检查器（validators）这些会在控制层进行介绍。

接下来定义`UserService`接口，在`service`目录下，定义`user.service.go`文件，内容如下
```go
package service

import "github.com/prithuadhikary/user-service/model"

type UserService interface {
   Signup(request *model.SignupRequest) error
}
```

接下来进行实现，[代码如下](https://gist.githubusercontent.com/prithuadhikary/2d69d1af160c46e6c06bc5839686f179/raw/f9cc44b18d7a78026b8afddba801ca19a783fd68/user.service.go)
```go
type userService struct {
   repository repository.UserRepository
}

func (service *userService) Signup(request *model.SignupRequest) error {
   if request.Password != request.PasswordConfirmation {
      return errors.New("password and confirm password must match")
   }
   exists := service.repository.ExistsByUsername(request.Username)
   if exists {
      return errors.New("email already exists")
   }
   service.repository.Save(&domain.User{
      Username: request.Username,
      Password: request.Password,
      Role:     "END_USER",
   })
   return nil
}

func NewUserService(repository repository.UserRepository) UserService {
   return &userService{
      repository: repository,
   }
}
```

这段代码也不难理解，
1. 比较密码是否相等
2. 进一步比较邮箱是否注册过了
3. 如果发生异常，则返回error

go语言中，error是一个包含 `Error() string`的空接口，定义在go的标准库中。一般在middleware使用一些自定义属性的error，会包含额外的属性。

有一个新方法如下：
```go
service.repository.ExistsByUsername(request.Username)
```
差点忘记了，我放在这里
```go
func (repository *userRepository) ExistsByUsername(username string) bool {
   var count int64
   repository.db.Model(&domain.User{}).Where("username = ?", username).Count(&count)
   return count > 0
}
```
这里利用GORM的属性，执行了一个数据库的query，注意，query中，Count方法传入了一个int64的指针，然后方法内写入了value。

东扯西扯，写了太多东西了，快速写一下Controller的实现
## 最终 The User Controller 控制层

接口文件如下，在`user-service/controller/user.controller.go` 中。
```go
package controller

import "github.com/gin-gonic/gin"

type UserController interface {
   Signup(ctx *gin.Context)
}
```

Signup方法中，入参是`gin.Context`的指针，这个指针的定义如下
```go
type HandlerFunc func(*Context)
```
``
`HanderFunc` ->  Handler Function！定义了处理来自HTTP请求的方法。这个方法一般会在路由中定义，作为某个路径的callback方法。will get invoked upon receiving an HTTP request at that route.

看下这个方法的[实现如下](https://gist.githubusercontent.com/prithuadhikary/a1f501aef142f3169a1487894f4ed07d/raw/f4127a98a204c5f3d16a829400c7db9bf7468282/user.controller.go)
```go
type userController struct {
   service service.UserService
}

func (controller userController) Signup(ctx *gin.Context) {
   request := &model.SignupRequest{}
   if err := ctx.ShouldBind(request); err != nil {
      ctx.AbortWithStatusJSON(http.StatusBadRequest, gin.H{
         "message": "Validation failed",
      })
      return
   }
   err := controller.service.Signup(request)
   if err != nil {
      ctx.AbortWithStatusJSON(http.StatusNotAcceptable, gin.H{
         "message": err.Error(),
      })
   }
}

func NewUserController(engine *gin.Engine, userService service.UserService) {
   controller := &userController{
      service: userService,
   }
   api := engine.Group("api")
   {
      api.POST("users", controller.Signup)
   }
}
```

方法中，使用了`ctx.ShouldBind(request)`，将request的body绑定在这个对象上`model.SignupRequest`对象上。

`ShouldBind`方法：会根据请求中的`Content-Type` 字段选择合适的反序列化策略，将请求的body进行解析，根据 `binding tags` 进行参数的校验。 在本地案例中，`ShouldBind` 发现入参的`Content-Type=application/json` 的话，会将传入的JSON结构绑定到 `model.SignupRequest` 结构的那个指针中。 现在，可以增加一些 `binding`标签了，如下：

快乐的绑定环节来了 **SignupRequest** [结构](https://gist.githubusercontent.com/prithuadhikary/9a28ffab6fce36785511e5d2f4df6310/raw/56f67a732bb2b1304a7aff979446395ee519cd6d/signup-reques.go)

```go
package model

type SignupRequest struct {
   Username             string `json:"username" binding:"required,email"`
   Password             string `json:"password" binding:"required,min=6"`
   PasswordConfirmation string `json:"passwordConfirmation" binding:"required"`
}
```
例子中，json的tag告诉gin，要从json的哪些字段中提取信息，`binding`的tags会指定了绑定过程中的一些约束，不满足约束，绑定会失败。回到Signup的方法实现中，绑定成功之后，会进入到service层调用的方法。如果绑定失败，则会返回异常。
`NewUserController` 方法也很直接
1. 接收参数，给下游调用。
2. 创建路由，创建一个 `gin.RouterGroup` 将`/api/users` 映射到`userController.Sigup` 这个方法上。后者被称为 handler方法。注意，`api`是一个group，`users`是一个接口。


## 优化参数校验结果返回

`ShouldBind`方法返回了 `validator.ValidationErrors` 的标准实现。通过遍历这个结构，可以拿到包含具体验证失败字段以及异常。如果前端返回的不明不白，类似`Validation failed` 这样的响应，其实调用者不知道如何改进

看下来自util的一个函数 [function:](https://gist.githubusercontent.com/prithuadhikary/6f704d6f355ae1163276451e8e718c61/raw/3c81496e382ed6ebd5bea3d46bb070f5cfd28916/render-binding-errors.go)

```go
package util

import (
// ... Skipping
)

type responseErr struct {
   Field     string `json:"field"`
   Condition string `json:"condition"`
}

func RenderBindingErrors(ctx *gin.Context, validationError validator.ValidationErrors) {
   var responseErrs []responseErr
   for _, fieldError := range validationError {
      field := fieldError.Field()
      responseErrs = append(responseErrs, responseErr{
         Field:     strings.ToLower(field[:1]) + field[1:],
         Condition: fieldError.ActualTag(),
      })
   }
   ctx.AbortWithStatusJSON(http.StatusBadRequest, responseErrs)
}
```

这段代码返回了400异常，遍历了异常集合，然后将所有异常的属性都增加在一个结构体内，返回客户端

把这段逻辑增加到controller层中如下：

```go
func (controller userController) Signup(ctx *gin.Context) {
   request := &model.SignupRequest{}
   if err := ctx.ShouldBind(request); err != nil && errors.As(err, &validator.ValidationErrors{}) {
      util.RenderBindingErrors(ctx, err.(validator.ValidationErrors))
      return
   }
   err := controller.service.Signup(request)
   if err != nil {
      ctx.AbortWithStatusJSON(http.StatusNotAcceptable, gin.H{
         "message": err.Error(),
      })
   }
}
```
`errors.As` 是一个内置方法，用来判定`err`的类型。如果 参数`err` 和 `&validator.ValidationErrors{}` 类型相同，会返回true。可以阅读下面的文档，了解更多error包下的内容： [documentation](https://pkg.go.dev/errors).

## 代码启动

这是启动的主方法。

```go
func main() {
   db, err := InitialiseDB(&DbConfig{
      User:     "postgres",
      Password: "password",
      DbName:   "groot",
      Host:     "localhost",
      Port:     "5432",
      Schema:   "users",
   })
   if err != nil {
      panic(err)
   }
   err = db.AutoMigrate(&domain.User{})
   if err != nil {
      panic(err)
   }

   userRepository := repository.NewUserRepository(db)
   userService := service.NewUserService(userRepository)

   engine := gin.Default()

   controller.NewUserController(engine, userService)

   log.Fatal(engine.Run(":8080"))
}
```

从底向上，一步一步构建，逻辑如下：

- 初始化GORM，连接数据库gostgres，获得`grom.DB`
- 调用`AutoMigrate`自动化适配表结构，升级数据库。
- 通过构造方法`NewUserRepository` 创建持久层，传入`gorm.DB` 
- 通过构造方法`NewUserService` 创建服务层，并且传入持久层对象`repository`。
- 使用默认配置创建`gin.Engine`，并且通过控制层构造方法`NewUserController` 创建控制层实例，将`gin.Engine`、`userService`传入 
- 启动gin，监听8080端口

日志如下:

```
...  
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.  
 - using env:   export GIN_MODE=release  
 - using code:  gin.SetMode(gin.ReleaseMode)**[GIN-debug] POST   /api/users                --> github.com/prithuadhikary/user-service/controller.userController.Signup-fm (3 handlers)**  
[GIN-debug] [WARNING] You trusted all proxies, this is NOT safe. We recommend you to set a value.  
**[GIN-debug] Listening and serving HTTP on :8080**
```

截图:
![](http://img.skydrift.cn/1725619256.png?imageMogr2/thumbnail/!70p)
调用curl，获得响应。

终于搞定，大工程，希望对你有帮助。

Github 库: [https://github.com/prithuadhikary/user-service](https://github.com/prithuadhikary/user-service)

byebye!

![](http://img.skydrift.cn/1725619271.png?imageMogr2/thumbnail/!70p)
