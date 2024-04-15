---
title: 项目开发-swagger篇
tags: [swagger]
date: 2023-10-18 10:07:00
categories: technique
urlname: 22
---

### OpenAPI规范


[OpenAPI 规范][1]（OpenAPI Specification），是定义一个标准的、与具体编程语言无关的RESTful API的规范。OpenAPI 规范可以在“不接触任何程序源代码和文档、不监控网络通信”的情况下理解一个服务的作用。遵循 OpenAPI 规范来定义 API，以使用文档生成工具来展示。主要的Swagger工具包括：

- Swagger Editor：基于浏览器的编辑器，可以在其中编写 OpenAPI 定义。
- Swagger UI：将 OpenAPI 定义渲染为交互式文档。
- Swagger Codegen：根据 OpenAPI 定义生成服务器存根和客户端库。
- Swagger Editor Next (beta)：基于浏览器的编辑器，可以在其中编写和审查 OpenAPI 和 AsyncAPI 定义。
- Swagger Core：用于创建、使用和处理 OpenAPI 定义的 Java 相关库。
- Swagger Parser：用于解析 OpenAPI 定义的独立库
- Swagger APIDom：为描述各种描述语言和序列化格式的 API 提供单一、统一的结构。


### Swagger
[Swagger][2] 是一套围绕 OpenAPI 规范构建的开源工具，可帮助开发人员设计、构建、记录和使用 REST API，支持文档在线自动生成以及接口功能测试。基于OpenAPI规范编写注解或通过扫描代码去生成注解，就能生成统一标准的接口文档和一系列 [Swagger 工具][3]。

以下以[go swagger][4]为例，说明如何为代码自动生成接口文档。

**环境准备**

> **安装swgger**

`go install github.com/swaggo/swag/cmd/swag@latest`

> **支持的web框架**

```
gin
buffalo
net/http
...
```

**swagger使用**

接口文档的生成过程：首先按照规范给接口代码（controller层）添加声明式注释，接着使用swagger工具扫描代码自动生成api接口文档数据，然后渲染在线接口文档页面。 

> **添加注释**

注释规范说明

![api declarative comments format][5]
~~~shell
@Summary      摘要
@Description  详细的说明
@Tags         每个API操作的标签列表，以逗号分隔。
@Accept       API可以使用的MIME类型列表，只影响具有请求正文的操作，例如POST、PUT、PATCH
@Produce      可以产生的 MIME 类型列表，MIME 类型可以简单的理解为响应类型，例如：json、xml、html 等等
@Param        参数格式，从左到右分别为：参数名、入参类型、数据类型、是否必填、注释
@Success      响应成功，从左到右分别为：状态码、参数类型、数据类型、注释
@Failure      响应失败，从左到右分别为：状态码、参数类型、数据类型、注释
@Router       路由，从左到右分别为：路由地址，HTTP 方法
~~~

具体例子
```
// ShowAccount godoc
//
//	@Summary		Show an account
//	@Description	get string by ID
//	@Tags			accounts
//	@Accept			json
//	@Produce		json
//	@Param			id	path		int	true	"Account ID"
//	@Success		200	{object}	model.Account
//	@Failure		400	{object}	httputil.HTTPError
//	@Failure		404	{object}	httputil.HTTPError
//	@Failure		500	{object}	httputil.HTTPError
//	@Router			/accounts/{id} [get]


// UploadAccountImage godoc
//
//	@Summary		Upload account image
//	@Description	Upload file
//	@Tags			accounts
//	@Accept			multipart/form-data
//	@Produce		json
//	@Param			id		path		int		true	"Account ID"
//	@Param			file	formData	file	true	"account image"
//	@Success		200		{object}	controller.Message
//	@Failure		400		{object}	httputil.HTTPError
//	@Failure		404		{object}	httputil.HTTPError
//	@Failure		500		{object}	httputil.HTTPError
//	@Router			/accounts/{id}/images [post]
```

> **生成接口文档**

在包含main.go文件的项目根目录运行`swag init`，解析注释并生成需要的文件（`docs`文件夹和`docs/docs.go`）

~~~shell
# swag 生成命令
swag init . 
~~~

如果部分接口不需要生成接口文档，则使用排除命令。
~~~shell
# swag 排除命令
swag init -d ./ --exclude ./app/api/v2
~~~


确保导入了生成的`docs/docs.go`文件，这样特定的配置文件才会被初始化。
```
//注册路由处
import (
   _ "xxx/docs"   //导入生成的docs
)
```



(可选) 可以使用`fmt`格式化 SWAG 注释。

```
swag fmt
```


> **渲染文档数据**

注册swagger api相关路由，可以自定义文档格式，也可使用现有工具的渲染，例如gin-swagger的swaggerFiles.Handler
```
	swagger := e.Group("swagger", middleware.SwaggerBasicAuth())
	{
		swagger.GET("/*any", api.Swagger.HandleReDoc)
	}
```

项目运行后，浏览器访问定义的文档地址即可查看


> **Swag cli**

~~~shell
swag init -h
NAME:
swag init - Create docs.go

USAGE:
swag init [command options] [arguments...]

OPTIONS:
--generalInfo value, -g value          API通用信息所在的go源文件路径，如果是相对路径则基于API解析目录 (默认: "main.go")
--dir value, -d value                  API解析目录 (默认: "./")
--exclude value                        解析扫描时排除的目录，多个目录可用逗号分隔（默认：空）
--propertyStrategy value, -p value     结构体字段命名规则，三种：snakecase,camelcase,pascalcase (默认: "camelcase")
--output value, -o value               文件(swagger.json, swagger.yaml and doc.go)输出目录 (默认: "./docs")
--parseVendor                          是否解析vendor目录里的go源文件，默认不
--parseDependency                      是否解析依赖目录中的go源文件，默认不
--markdownFiles value, --md value      指定API的描述信息所使用的markdown文件所在的目录
--generatedTime                        是否输出时间到输出文件docs.go的顶部，默认是
--codeExampleFiles value, --cef value  解析包含用于 x-codeSamples 扩展的代码示例文件的文件夹，默认禁用
--parseInternal                        解析 internal 包中的go文件，默认禁用
--parseDepth value                     依赖解析深度 (默认: 100)
--instanceName value                   设置文档实例名 (默认: "swagger")
~~~


~~~shell
swag fmt -h
NAME:
   swag fmt - format swag comments

USAGE:
   swag fmt [command options] [arguments...]

OPTIONS:
   --dir value, -d value          API解析目录 (默认: "./")
   --exclude value                解析扫描时排除的目录，多个目录可用逗号分隔（默认：空）
   --generalInfo value, -g value  API通用信息所在的go源文件路径，如果是相对路径则基于API解析目录 (默认: "main.go")
   --help, -h                     show help (default: false)

~~~






[1]: https://openapi.apifox.cn/
[2]: https://swagger.io/
[3]: https://github.com/swagger-api
[4]: https://github.com/swaggo/swag
[5]: https://cdn.jsdelivr.net/gh/etamsylate-pupu/Image-host/blogImg/hexoBlog/API.png