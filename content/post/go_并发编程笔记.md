---
title: "Go_并发编程笔记"
date: 2021-02-08T11:15:50+08:00
categories:
- go
- 学习笔记
tags:
- go
- 并发编程
keywords:
- go
- 编发
thumbnailImage: /img/xlzo5o.jpg
autoThumbnailImage: true
thumbnailImagePosition: "left"
coverImage: /img/4gj39d.jpg
metaAlignment: center
coverMeta: out
---

<!--more-->
# go 并发编程学习笔记
## go命令

> ```build``` 命令源码编译成可执行文件

> ```clean``` 清理因执行go命令而遗留下来的临时目录和文件

> ```doc``` 用于显示go语言代码包和程序实体的文档

> ```env ``` 打印go语言相关环境变量

>``` fix``` 用于修正指定的代码包的源码文件中包含的过时语法和代码调用

>```fmt``` 用于格式化指定代码包的go源码文件 实际是通过gofmt命令实现的功能

>``` generate``` 用于识别代码中go:generate注释，并执行其携带的命令

> ``` get ```用于下载编译并安装指定的代码包和其依赖包

> ``` install ``` 用于安装指定的代码包和其依赖包

> ``` list``` 用于显示指定代码包的信息

> ``` run``` 用于编译并运行指定的命令源码文件

> ``` test``` 用于测试指定的代码包，前提是代码包包含测试文件

> ``` tool``` 用于运行go语言的特殊工具 其中包括 addr2line 、api、asm、buildid、cgo、compile、cover、dist、doc、fix、link、nm、objdump、oldlink、pack、pprof、test2json、trace、vet 其中pprof可以分析cpu、内存和程序阻塞概要文件，trace 读取程序踪迹文件并以图形化展示

>``` vet``` 用于检查指定代码包的go语言源码，并报告可疑代码问题

> ``` version``` 用于显示当前的go语言版本

> ``` -a ```用于强制编译所有设计的go语言的代码包

> ``` -n``` 只打印命令而不执行

> ``` -race``` 检测指定go语言程序中存在的数据竞争问题

> ``` -v ```用于检测命令执行中设计的代码包

>``` -work``` 打印命执行时生成的临时目录的名字，且命令执行后不删除他们

>``` -x``` 使命令打印执行过程中用到的所有命令，并执行他们

## 一个小demo

> 
```package main
import (
	"bufio"
	"fmt"
	"os"
)
func main()  {

	reader := bufio.NewReader(os.Stdin)
	fmt.Println("请输入您的名字")
	readString, err := reader.ReadString('\n')
	//line, _, err := reader.ReadLine()
	if err != nil {
		fmt.Println("输入错误")
	}
	//fmt.Println("你好,",string(line))
	fmt.Println("你好,",readString[:len(readString)-1])
}
```

## go的内建函数名称

>append、cap、close、complex、copy、delete、imag、len、make、new、panic、print、real、recover

## go的关键字

### 程序声明

>import、package

### 实体声明和定义

>chan、const、func、interface、map、struct、type、var

### 流程控制

> go、select、break、case、continue、default、else、fallthrough、for、goto、if、range、return、swith

## 表达式

> ```v1.(I1)``` 类型断言

> 注意： 失败的断言会引起panic 所有通常赋值两个变量即``` var i1,ok = v1.(I1) ``` 

## switch 类型语句

> 
``` 
switch v.(type) 

```
> 该操作不能使用fallthrough

## for 语句的省略问题

> 当初始化语句和后置语句都省略，或者全部都省略的时候可以不写；

## range 迭代时注意事项

> 数组、切片、字符串进行迭代时，当只有一个变量时，此时是索引

> 迭代空数组、空字符串、nil的切片、nil的字典时，并不会执行for中的代码

> 迭代nil的chan时会使当前流程阻塞在for语句上

## 关于flag的注意事项
> 代码如下
```
	var str string
	flag.StringVar(&str, "str", "123", "输入字符串")
	flag.Parse()//一定要指定该语句 否则赋值将不会成功
	fmt.Println(str)
```

## 并发编程【重要，将不断完善】

> 进程的状态转换

![demo](/img/status.png)

> 内核态和用户态 ：cpu一办只工作再用户态，当需要进行系统调用（访问内核的接口），内核会将cpu由用户态调整到内核态，这时候的cpu就有权限访问内核空间，在执行完对应的函数时，内核将cpu从内核态切换成用户态

![demo](/img/thread.png)

> 线程的状态

![demo](/img/thread_model.png)
>线程模型
