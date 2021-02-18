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
```go
package main
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
```go 
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
```go
	var str string
	flag.StringVar(&str, "str", "123", "输入字符串")
	flag.Parse()//一定要指定该语句 否则赋值将不会成功
	fmt.Println(str)
```

## 并发编程【重要，将不断完善】

> 进程的状态转换

![demo](/img/status.png)

> 内核态和用户态 ：cpu一办只工作再用户态，当需要进行系统调用（访问内核的接口），内核会将cpu由用户态调整到内核态，这时候的cpu就有权限访问内核空间，在执行完对应的函数时，内核将cpu从内核态切换成用户态

> 线程的状态如下
![demo](/img/thread.png)

>线程模型如下
![demo](/img/thread_model.png)

>处理并发的建议
![demo](/img/suggest.png)


## go中的M、P、G

>M：一个M代表一个内核线程，或者工作线程

>P:一个P代表一个GO代码段所必须的资源(上下文)

>G:go代码段 代表一个goroutine

## runtime和goroutine包

>```runtime.GOMAXPROCS()``` 设置P的最大数 但是超过256则也只能使用256

>```runtime.Goexit()``` 终止当前goroutine，其他goroutine不受影响，但是终止前会指定当前doroutine未执行的defer

>```runtime.Gosched()``` 暂停当前的goroutine，等调度器再次将当前goroutine唤醒

>```runtime.NumGoroutine()``` 打印当前非dead的G个数，该值永远大于等于1

>```runtime.LockOSThread()``` 将当前的goroutine和M绑定在一起

>```runtime.UnlockOSThread()``` 将goroutine和M解绑，单独执行绑定和解绑不会产生其他不好的结果

>```debug.SetMaxStack()``` 设置单个goroutine所能申请的栈大小 32和64位系统设置默认值为250m和1G

> ```debug.SetMaxThreads()``` 设置内核线程的数量（也可以认为是M的数量） 默认值是10000

## 关于 channel

### 建议
>建议使用传递信号的channel都使用struct{}空结构体，因为空结构体变量不占用内存空间，所有空结构体的内存地址相同

> 环形队列

![demo](/img/queue-chan.png)

### 发送和接受通道的记忆方式

> 可以使用顺序记忆法 out<-chan<-in 方式记忆通道类型 chan<- 则为发送通道，理解为从右往chan输入  <-chan 则为接受 即根据chan和<-的方向记忆

### 关于发送和接受通道的使用 

><-chan 接收通道对于定义来说毫无意义，想象下定义一个只能接收的通道，又不能发送，那接收就没有意义，其实这种通道对于方法或者函数参数的定义很有意义，一个只能接收的通道的参数标识这该参数只能接收数据，虽然可以传递一个既可以发送也可以接收的通道，但是参数的定义类型标识着通道的试用方法。

## 非缓冲通道

![demo](/img/happen_before.png)

> 对于非缓冲通道的使用【定时器】
```go
	timer := time.NewTimer(3 * time.Second)
	c := <-timer.C
	fmt.Println("%v",c)
```
如果没有到定时器的时间，则线程一直会阻塞着

如果想直接返回接收通道 可以使用after函数
```go
	after := time.After(4 * time.Second)
	d:=<-after
	fmt.Println("%v",d)
```

## api请求超时
![demo](/img/api.png)