---
title: "go-zero教程"
date: 2022-03-21T14:49:20+08:00
categories:
- 教程
- go-zero
tags:
- go
- go-zero
- 教程
keywords:
- tech
thumbnailImage: /img/Hippopx-03.webp
autoThumbnailImage: true
thumbnailImagePosition: "left"
coverImage: /img/wqve97.webp
metaAlignment: center
coverMeta: out
---

<!--more-->

# go-zero 手把手教程

## 安装go 已安装的跳过  
``` go version  ``` 查看当前的go版本

## 安装goctl 工具 
``` go install github.com/zeromicro/go-zero/tools/goctl@latest ``` 注意需要配置加速 才能下载下来 

## 创建项目目录 
``` mkdir study ```

``` cd study ```

```go mod init study```

## 创建api目录

``` mkdir api ```

``` cd api```

``` goctl api -o study.api ```

## 根据api定义的接口生成handler 等文件 

``` goctl api go -api study.api -dir ../ ```

## 下载依赖

在项目根目录下执行  ``` go mod tidy ``` 下载对应的依赖

## api文件如下
``` 
type HomeRequest {

	// 用户id
	UserId string `form:"userId"`
}

type HomeResponse {

	// 已读页码
	Pages int8 `json:"pages"`
	// 上次阅读时间
	LastReadTime int64 `json:"lastReadTime"`
}

service study-api {
	
	// 获取首页内容
	@handler HomeHandler
	get /home(HomeRequest) returns (HomeResponse)
}
```

## 创建model 目录

和api 同级创建model目录存放实体相关代码

## 添加sql文件并生成对应的代码

``` goctl model mysql ddl -src user.sql -dir . -c ``` 


## 添加数据库配置 
在 etc目录下编辑yaml文件 添加需要的mysql和rdis 相关配置 

```
Mysql:
  DataSource: $user:$password@tcp($url)/$db?charset=utf8mb4&parseTime=true&loc=Asia%2FShanghai
CacheRedis:
  - Host: $host
    Pass: $pass
    Type: node 
```

## 修改config/config.go 文件

添加如下代码

```
Mysql struct{
        DataSource string
    }

    CacheRedis cache.CacheConf
 ```

## 修改svc中的相关代码
注意 redis mysql 相关的配置 是通过servicecontext进行传递的 
ServiceContext中添加 model

``` 
type ServiceContext struct {
	Config config.Config  
	UserModel model.StudyUserModel // 新增代码 新增的实体类 都需要进行配置
}

func NewServiceContext(c config.Config) *ServiceContext {
	conn := sqlx.NewMysql(c.Mysql.DataSource) // 新增代码 mysql 配置
	return &ServiceContext{
		Config: c,
		UserModel: model.NewStudyUserModel(conn,c.CacheRedis),// 新增的代码
	}
}

```