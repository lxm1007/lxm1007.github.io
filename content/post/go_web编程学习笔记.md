---
title: "Go_web编程学习笔记"
date: 2021-02-05T20:22:34+08:00
categories:
- go
- 学习笔记
tags:
- go
- web
- 学习笔记
keywords:
- go
thumbnailImage: /img/5wkvz8.webp
autoThumbnailImage: true
thumbnailImagePosition: "left"
coverImage: /img/oxxw29.jpg
metaAlignment: center
coverMeta: out
---

<!--more-->

# go web编程学习笔记
## 【书籍github地址：https://github.com/sausheong/gwp】

# 关于http的url encode

> 所有保留字符都要进行url编码：把保留字符转化为该字符的ascii编码中对应的十六进制对应的字节值，然后在该字节前面加%，例如空格的asscii码对应的十六进制为20 则urlencode后为%20

# 简单的hello world 

> ![demo](/img/demo.webp)

# 复杂的应用 

> 
```go
mux := http.NewServeMux() //创建多路复用器
files := http.FileServer(http.Dir("/public"))//创建一个可以为指定目录服务的文件处理器
mux.Handle("/static/", http.StripPrefix("/static/", files)) //以static开头的文件 去掉/static/ 到public 下寻找对应文件 
```


## 处理器函数

```go
package main

import (
	"fmt"
	"net/http"
)

func hello(w http.ResponseWriter, resuest *http.Request)  {
	fmt.Fprintf(w,"hello world")
}

func test(w http.ResponseWriter,request *http.Request){
	fmt.Fprintf(w,"test")
}

func main() {
	//go拥有一种 handleFunc 函数类型，他可以把一个带有正确函数签名的转化成一个带有方法f的Handler
	server := http.Server{Addr: "127.0.0.1:8080"}
	http.HandleFunc("/hello",hello)
	http.HandleFunc("/test",test)
	server.ListenAndServe()
}

//如果不使用handleFunc 需要使用下面这种方法
//type helloHandler struct {}
//
//func (hello *helloHandler)hello(w http.ResponseWriter,request *http.Request){
//	fmt.Fprintf(w,"hello")
//}
//type testHandler struct {}
//
//func (hello *testHandler)test(w http.ResponseWriter,request *http.Request){
//	fmt.Fprintf(w,"test")
//}
```

## 最小惊讶原则

>
也称最小意外原则：它指的是我们在进行设计的时候应该做那些合乎常理的事情使事物的行为始终显而易见始终如一并且合乎情理，举例：我们在一扇门旁边放置一个按钮，那么我们认为这个按钮应该和这个门有关系，比如门铃，而不是按下去之后打开走廊灯

## 链式处理（中间件）

>
```go
package main

import (
	"fmt"
	"net/http"
)

func hello(w http.ResponseWriter,req *http.Request){
	fmt.Fprint(w,"test")
}
//将hello这个handleFunc 包装，并且添加自己想要的逻辑
func log(handlerFunc http.HandlerFunc) http.HandlerFunc{
	return func(writer http.ResponseWriter, request *http.Request) {
		fmt.Println("hahahah")
		handlerFunc(writer,request)
	}
}

func main()  {
	server := http.Server{Addr: "127.0.0.1:8080"}
	http.HandleFunc("/test",log(hello))
	server.ListenAndServe()
}
```

## 最小惊讶原则应用的路由问题

>
```go
package main

import (
	"fmt"
	"net/http"
)
func hello1(w http.ResponseWriter,req *http.Request){
	fmt.Fprint(w,"test1")
}

func hello2(w http.ResponseWriter,req *http.Request){
	fmt.Fprint(w,"test2")
}
func main()  {
	server := http.Server{Addr: "127.0.0.1:8080"}
    //因为结尾没有/ 所以/test/* 这种请求将不会执行hello1 
	http.HandleFunc("/test",hello1)
    ///test/*类型的请求则会执行hello2 
	http.HandleFunc("/test/",hello2)
	server.ListenAndServe()
}
```

## go语言开发技巧

> GO111MODULE 在mod模式下下载的包不在$GOPATH/src下 而是在pkg/mod下 所以导入第三方包的时候要注意 在编译运行等命令前加 ```GO111MODULE=off``` 或者将全局的开关关闭 但是不要忘了mod项目的时候打开

> httprouter 
```go
package main

import (
	"fmt"
	"github.com/julienschmidt/httprouter"
	"net/http"
)

func hello(w http.ResponseWriter ,request *http.Request,params httprouter.Params)  {
	fmt.Fprintf(w,params.ByName("name")+"\n")
	fmt.Fprint(w,"hello world")
}

func main(){

	router := httprouter.New()
	router.GET("/test/:name",hello)
	server := http.Server{Addr: "127.0.0.1:8080",Handler: router}
	server.ListenAndServe()
}
```
## form中取值的各个方法以及能获取的值

![](/img/form.webp)

## 关于ResponseWriter的三个方法使用

>```Header().set``` 进行响应头的设置

> ```WriteHeader(http status code) ``` 设置响应状态码

> ```Write``` 设置返回内容

> 需要注意的是在设置状态码之后再进行设置响应头则无法设置成功，所以响应头要在WriteHeader之后设置

## 模板解析

```go
package main

import (
	"html/template"
	"net/http"
)

func test11(w http.ResponseWriter,r *http.Request){
	//files, _ := template.ParseFiles("1.html")
	//files.Execute(w,"hello ")
    //此时返回的是一个模板集合 则再execute时 默认是第一个 想要指定的模板execute时 使用ExecuteTemplate 指定模板名即可
	parseFiles, _ := template.ParseFiles("1.html","2.html")
	datas := make(map[string]string,2)
	datas["name"] = "lxm"
	datas["age"]="12"
	parseFiles.ExecuteTemplate(w,"2.html",datas)
}

func main()  {

	server := http.Server{Addr: "127.0.0.1:8080"}
	http.HandleFunc("/test",test11)
    server.ListenAndServe()

}
```
# {{ }}语法

> {{ . }} 获取所有的数据 {{ .data }} 获取所有数据中的data字段值

> {{ range . }} 循环遍历数据 {{ end }} 结束 
```go
{{ range . }}
{{ . }}
{{ end }}
```

>设置动作 with
```go
{{ with "test"}}
{{ . }}
{{ end }}
```

>引入模板
```go
{{ template "1.html"}}
```
想要在使用引入其他的模板并且传递参数时，可以使用 {{ template "1.html" args }} 其中args是想要传递的数据 这样在1.html中的数据就可以进行解析

>定义模板
```go
{{ define "layout" }}
    这中间是layout模板的内容 一个模板文件可以定义多个不同名的模板
{{ end }}
```



