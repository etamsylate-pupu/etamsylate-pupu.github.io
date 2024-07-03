---
title: gin参数重复绑定 | Go
tags: [gin]
date: 2024-07-03 16:31:33
categories: technique
urlname: 39
---

使用Gin框架时，考虑一种场景：在自定义的中间件里面获取相关的参数进行参数解析和绑定，在路由API的内部也提取参数进行绑定。那么gin提供多次绑定同一参数的方法吗？


首先查看下面的代码[gin对请求体类型为json的参数两次绑定](https://www.nowcoder.com/feed/main/detail/8c593266724d48f990e3e25e7fd49c49?sourceSSR=users)，使用c.ShouldBindJSON()方法，读取两次请求体的数据流，绑定至名为json的对象。
```
import (
	"fmt"
	"net/http"

	"github.com/gin-gonic/gin"
)
type Foo struct {
	Name string `json:"name" form:"name"`
	Age  int    `json:"age" form:"age"`
}
func main() {
	r := gin.Default()

	// GET endpoint
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{
			"message": "pong",
		})
	})

	// POST endpoint
	r.POST("/foo", func(c *gin.Context) {
		var json Foo
		if err := c.ShouldBindJSON(&json); err != nil {
			c.JSON(http.StatusBadRequest, gin.H{
				"error": err.Error(),
			})
			return
		}
		fmt.Printf("first: %v\n", json)
		if err := c.ShouldBindJSON(&json); err != nil {
			fmt.Println("Error:", err)
			c.JSON(http.StatusBadRequest, gin.H{
				"error": err.Error()})
			return
		}
		fmt.Printf("second: %v\n", json)

		// Process the data
		c.JSON(http.StatusOK, gin.H{
			"name": json.Name,
			"age":  json.Age,
		})
	})
	// Start the server on 0.0.0.0:8080
	r.Run()

}
```


构造请求体
```
{
	"name": "Alice",
	"age": 12
}
```

请求`http://127.0.0.1:8080/foo`接口，终端输出`EOF (End Of File)`错误：
```
[GIN] 2024/07/03 - 16:47:47 | 400 |      1.2247ms |       127.0.0.1 | POST     "/foo"
first: {Alice 12}
Error: EOF
```

【原因】：在 Gin 框架中，使用`application/json`格式的参数请求POST接口时，多次调用 `ShouldBind(&data)` 或 `ShouldBindJSON(&data)`方法，使用了c.Request.Body，会导致c.Request.Body(请求体的数据流)被读取多次。由于gin的Context中的Request.Body对象在第一次调用ShouldBind / ShouldBindJSON 方法后已经被读取完毕，再次调用时已经到达了EOF，导致无法再次读取请求体。


那么，使用`application/form-data`格式的参数调用POST接口，结构体增加`form`映射，多次调用 `ShouldBind()`方法，结果如何？
```
[GIN] 2024/07/03 - 17:21:44 | 200 |       4.942ms |       127.0.0.1 | POST     "/foo"
first: {Alice 12}
second: {Alice 12}
```

可以发现，Gin 框架可以正常处理多次读取并绑定。原因如下[1341](https://github.com/gin-gonic/gin/pull/1341)：

> Other formats, query & forms, are already available to be called multiple times, because they uses parsed req.Form (url.Values) instead of req.Body.


如何做到`application/json`格式的请求体被多次绑定呢？ 可以使用`ShouldBindBodyWith(&data,binding.JSON)`方法。`ShouldBindBodyWith()`会在绑定之前将 c.Request.Body 存储到上下文中（这会对性能造成轻微影响，如果调用一次就能完成绑定的话，那就不要用这个方法，只有某些格式需要此功能，如 JSON, XML, MsgPack, ProtoBuf），再次调用时，复用存储在上下文中的body。
```
if err := c.ShouldBindBodyWith(&json,binding.JSON); err != nil {
    c.JSON(http.StatusBadRequest, gin.H{
        "error": err.Error(),
    })
    return
}
fmt.Printf("first: %v\n", json)
if err := c.ShouldBindBodyWith(&json,binding.JSON); err != nil {
    fmt.Println("Error:", err)
    c.JSON(http.StatusBadRequest, gin.H{
        "error": err.Error()})
    return
}
fmt.Printf("second: %v\n", json)
```

输出为：
```
[GIN] 2024/07/03 - 17:41:09 | 200 |       502.2µs |       127.0.0.1 | POST     "/foo"
first: {Alice 12}
second: {Alice 12}
```

综上所述，当使用`application/json`格式的请求体调用POST接口时，使用`ShouldBind()` 或 `ShouldBindJSON()`会出现EOF错误，可以使用`ShouldBindBodyWith(&data,binding.JSON)`方法进行正常的读取。`ShouldBindBodyWith(&data,binding.JSON)`方法可以做类型判断兼容多种格式的请求体。
```
if ctx.ContentType() == "application/json" {
    return ctx.ShouldBindBodyWith(data, binding.JSON)
}else if ctx.ContentType() == "application/xml"{
    return ctx.ShouldBindBodyWith(data, binding.XML)
}

```
