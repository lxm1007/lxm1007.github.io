---
title: "Go_web编程学习笔记"
date: 2021-02-05T20:22:34+08:00
categories:
- go
- 学习笔记
tags:
- go
- web
- 笔记
keywords:
- go
thumbnailImage: /img/5wkvz8.png
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

> ![demo](/img/demo.png)

# 复杂的应用 

> 
```go
mux := http.NewServeMux() //创建多路复用器
files := http.FileServer(http.Dir("/public"))//创建一个可以为指定目录服务的文件处理器
mux.Handle("/static/", http.StripPrefix("/static/", files)) //以static开头的文件 去掉/static/ 到public 下寻找对应文件 ```

