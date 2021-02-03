近期在学习gin的时候发现对请求参数的校验很麻烦, 且重复代码很多, 进行一番思考和实践后发现了一种使用反射实现在 controller 函数上实现自动提取请求参数到指定的 struct, 并且自动使用 validation 进行校验.

## 起因

如下, 这是一段很普通的处理登录的代码, 获取请求参数, 验证参数是否正确在错误的时候返回错误码, 这种代码在项目中很常见, 很重复, 写这种代码总是让人厌烦.

```go
type LoginParam struct {
	Account  string `validate:"required" json:"account"`
	Password string `validate:"required" json:"password"`
}

func main(){
  g := gin.New()
  g.POST("login", LoginHandlerFunc)
  _ = g.Run(":8080")
}

// HanlderFunc
func LoginHandlerFunc(ctx *gin.Context) {
    param := LonginParam{}
    err := ctx.ShouldBind(&param)
    if err != nil {
       ctx.String(400, "请求参数异常")
       return
    }
		secc, msg := Validate(&param)
		if !secc {
			ctx.String(400, msg)
			return
    }
		// ... CRUD
}
```

以下是验证器, 省略了翻译器的注册等代码.

```go
import (
	github.com/go-playground/validator/v10"
)

var v = validator.New()

func Validate(param interface{}) (bool, string) {
	  err := v.Struct(param)
	  errs := err.(validator.ValidationErrors)
		if len(errs) > 0 {
			err := errs[0]
			return false, err.Translate(that.trans)
		}
		return true, ""
}
```

### 目标

映射参数和校验参数都是固定的, 目标是将 `LoginHandlerFunc` 优化成如下所示.

```go
func LoginHandlerFunc(ctx *gin.Context, params *LonginParam) {
	// ... CRUD
}
```

## 思路

**问题一: 如何根据函数参数实现自动创建**

我无法知道请求参数所映射 struct 的具体类型, 那只能使用反射, 可以反射获取到函数的参数, 具体实践如下.

```go
func reflectHandlerFunc(handlerFunc interface{}){
	funcType := reflect.TypeOf(handlerFunc)
	// 判断是否 Func
	if funcType.Kind() != reflect.Func {
		panic("the route handlerFunc must be a function")
	}
	// 获取第二个参数的类型
	typeParam := funcType.In(1).Elem()
	// 创建实例
	instance := reflect.New(typeParam).Interface()
}
```

如上, 通过反射 handlerFunc 然后获取函数的第二个参数即可拿到 `Type` 然后再通过反射创建实例.

**问题二: 如何映射参数并且验证**

针对问题一中的代码进行一下优化.

```go
func proxyHandlerFunc(ctx *gin.Context, handlerFunc interface{}){
	funcType := reflect.TypeOf(handlerFunc)
  funcValue := reflect.ValueOf(handlerFunc)

	// 判断是否 Func
	if funcType.Kind() != reflect.Func {
		panic("the route handlerFunc must be a function")
	}
	// 获取第二个参数的类型
	typeParam := funcType.In(1).Elem()
	// 创建实例
	param := reflect.New(typeParam).Interface()
	// 绑定参数到 struct
	err := ctx.ShouldBind(&param)
	if err != nil {
		ctx.String(400, "请求参数异常")
		return
	}
	// 验证参数
	succ, msg := Validate(&param)
	if !succ {
		ctx.String(400, msg)
		return
	}
	// 调用真实 HandlerFunc
	reflect.Call(valOf(ctx, param))
}

func valOf(i ...interface{}) []reflect.Value {
	var rt []reflect.Value
	for _, i2 := range i {
		rt = append(rt, reflect.ValueOf(i2))
	}
	return rt
}
```

如此, 就完成了一个对 HandlerFunc 的代理工作, 只要我们在注册路由时包装一下真实 HandlerFunc 即可.

```go

// ...
g.POST("login", getHandlerFunc(LoginHandlerFunc))
// ...

func getHandlerFunc(handlerFunc interface{}) func(*gin.Context) {
	return func(context *gin.Context){
		proxyHandlerFunc(context, handlerFunc)
	}
}

// ...

func LoginHandlerFunc(ctx *gin.Context, param *LoginParam){
		// ... CRUD
}
```

## 代码及性能优化

**0.兼容原有 HandlerFunc**

假设项目已经进行到了一半, 而我无法对所有 HandlerFunc 进行一步到位的重构, 则需要兼容原来的方法, 这个只需加一个简单的判断即可.

```go
func getHandlerFunc(handlerFunc interface{}) func(*gin.Context) {
	// 获取参数数量
	paramNum := reflect.TypeOf(handlerFunc).NumIn()
	valueFunc := reflect.ValueOf(handlerFunc)
	return func(context *gin.Context){
		// 只有一个参数说明是未重构的 HandlerFunc
		if paramNum == 1 {
				valueFunc.Call(valOf(context))
				return
		}
		proxyHandlerFunc(context, handlerFunc)
	}
}
```

**1.针对特定的参数进行手动绑定并且验证**

在实际开发中, 可能部分接口是表单, 有些接口是 JSON, 有些是其他类型, 以上代码只能由 `gin.ShouldBind` 自动处理绑定到 struct 的过程, 针对这个问题, 给 `Param` 实现特定接口即可, 如果实现了我们就是用该接口的方法进行绑定, 具体实现如下.

```go
type Deserialzer interface {
	DeserializeFrom(ctx *gin.Context) error
}

type LoginParam struct {
	Account  string `validate:"required" json:"account"`
	Password string `validate:"required" json:"password"`
}

func (that *LoginParam) DeserializeFrom(ctx *gin.Context) error {
		return ctx.ShouldBindWith(that, binding.FormPost)
}
```

经过以上改造, 我们将具体的绑定过程交给具体的 struct 自己, 对于所有实现了 `Dserializer` 接口的 struct 都进行自定义绑定, 之后只需要对 `proxyHandlerFunc` 进行一点小改动即可实现这个功能的适配.

```go
func proxyHandlerFunc(ctx *gin.Context, handlerFunc interface{}){
	// ...
	// 创建实例
	param := reflect.New(typeParam).Interface()
	deser, ok := param.(Deserialzer)
	// 如果未实现 Deserializer 接口则说明该 struct 使用默认绑定过程即可.
	if !ok {
		// 绑定参数到 struct
		err := ctx.ShouldBind(&param)
		if err != nil {
			ctx.String(400, "请求参数异常")
			return
		}
	} else {
		// 绑定请求参数
		err := deser.DeserializeFrom(ctx)
		if err != nil {
			ctx.String(400, "请求参数异常")
			return
		}
		param = reflect.ValueOf(deser).Interface()		
	}
	// 验证参数
	succ, msg := Validate(&param)
	// ...
}
```

对于自定义参数验证过程也是按一样的方法即可实现.

**性能优化**

GO 的反射对性能的影响是巨大的, 因此应尽量避免在 HandleFunc 中使用反射, 以上功能使用反射和不使用耗时相差约300倍. 所以, 部分 `Type`, `Value` 可以在注册路由时进行反射, 提前反射, 就避免了每次使用都反射. 
