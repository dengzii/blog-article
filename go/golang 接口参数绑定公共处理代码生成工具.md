很久之前写过一个自动创建, 校验请求参数的文章: gin 自动映射参数及自动校验, 文章介绍了一种通过反射创建请求参数, 绑定请求参数到实例, 校验处理, 并调用 HandlerFunc 的手段. 最近我想到用代码生成来做这个功能, 这样就没有反射带来的性能问题, 顺便玩一下 go 语言元编程.

使用该工具后, 接口处理函数将变成下面这样:

```go
func TestHandler(ctx *gin.Context, request *param.Request) (*param.Response, error) 
```

这个工具可以生成一个包装函数, 帮你做那些无聊的操作, 比如创建请求参数结构体, 绑定请求参数, 绑定响应, 校验参数等等之类的, 生成的包装函数再调用这个函数处理真正的业务, 设置路由的时候改成生成的包装函数即可.

## 安装

这个是一个工具, 不会依赖到项目, 如果出现找不到命令, 记得添加 `gopath/bin` 目录到环境变量.

```go
go install github.com/dengzii/genx@v1.1.0
```

## 效果

使用 genx 后, 我们的 API 处理函数就变成下面这样子了, 函数第二个参数就是请求绑定的结构体, 返回的分别是响应体和错误, 所有参数都是可选的, 但是顺序不可变, 可选指针, 例如没有请求参数则去除第二个参数即可.

```go
//go:generate genx handler
func TestHandler(ctx *gin.Context, request *param.TestRequest) (*param.TestResponse, error) {
// ...
return &param.LoginResponse{Token: "token"}, nil
}
```

只需要给函数添加一行注释, 注释没有要求必须是函数注释第几行, 注意 `//` 后面没有空格, goland 左侧就会出现一个运行按钮, 也可以在包目录下执行 `go generate` 命令或者 `genx`, 工具会扫描当前包下所有带有该注释的函数, 并且以文件做划分生成对应的绑定参数代码.

```go
//go:generate genx handler
```

生成后的代码大概如下

```go
func GenxTestHandler(ctx *gin.Context) {
   req := &param.LoginRequest{}
   _ = ctx.BindJSON(req)
   resp, err := TestHandler(ctx, req)
   if err != nil {
      ctx.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
      return
   }
   ctx.JSON(http.StatusOK, resp)
}
```

之后我们绑定路由的时候就在原函数前加一个 `Genx` 前缀即可

## 计划

目前只支持最基础的功能: 绑定参数, 校验什么的都还没有, 暂时只支持 `gin`, 不知道大家对这个小工具的看法如何, 欢迎在评论区发表你的看法. 


- [x] 生成 api handler func 绑定函数
- [x] 生成绑定 json 请求参数到结构体
- [x] 生成绑定 json 响应
- [ ] 支持定多种参数类型(Query, Form 等)
- [ ] 支持 gin 外的 web framework
- [ ] 自定义校验器支持
- [ ] 生成公共响应包装
- [ ] 参数校验
- [ ] 错误处理及自定义处理过程

github:  [https://github.com/dengzii/genx](https://github.com/dengzii/genx)

觉得不错的话 star 一下吧, 任何想法都可以在 issue 中提交.
