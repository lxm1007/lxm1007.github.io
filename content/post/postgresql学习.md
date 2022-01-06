---
title: "Postgresql学习"
date: 2020-12-06T17:28:25+08:00
categories:
- 数据库
- postgresql
tags:
- json
- postgresql
keywords:
- 数据库
thumbnailImage: /img/r2e391.jpg
autoThumbnailImage: true
thumbnailImagePosition: "left"
coverImage: /img/2020.webp
metaAlignment: center
coverMeta: out
---

<!--more-->

# Postgresql的json使用
因为一直很好奇，为什么网上说postgresql的速度比mysql要快很多，但是国内用的人却很少，况且，postgresql还支持json。
于是抱着试试看的态度用docker部署一个postgresql作测试。记录过程如下

* 部署postgresql `docker run -d -p 5432:5432  -v /my/own/datadir:/var/lib/postgresql/data -e POSTGRES_PASSWORD=123456 ` 因为不想持久化数据 所有就不挂载数据了
* 修改远程登录 `docker exec -it 732042e356e8 psql -U postgres -d postgres -h 127.0.0.1 -p 5432 `
* 创建数据表test_json 只需要一列数据 json_text数据格式为jsonb (关于json和jsonb的大概区别就是：jsonb在存的时候进行数据处理，包括重复键的覆盖，所以存慢取快，json保留存的时候的重复键，所以存快取慢，大部分应该都是使用存慢取快这种吧)
* 在对应的字段上加上索引`create index idx_json on test_json using gin (json_text)`
* `INSERT INTO test_json(json_text) VALUES('{"name":"lxm","age":12}');` 先插入一条数据 然后试用批量复制的方法创建200W 条数据 `INSERT INTO test_json(json_text) SELECT json_text from test_json`
* 查询 存在key的数据 `SELECT json_text->'name' as name,json_text->'age'as age from test_json  WHERE json_text ? 'name';` 查询json中存在name的key的数据 200w数据大概需要用时7s
* 查询value存在的数据 `SELECT json_text->'name' as name,json_text->'age'as age from test_json  WHERE json_text @> '{"age":12}';` 查询时间6.7s


# 基础教程

## 操作命令 
> ```\l``` 查询所有数据库

>```\d``` 不加参数时查询所有表

>```\q``` 退出

> ```\c 数据库名``` 切换到postgres数据库

>```psql -U postgres -h 127.0.0.1 -p 5432``` 连接到postgres服务

>```\d 表名``` 显示表结构的详细信息

>```\dt ```只显示表的信息 ```\di``` 只显示索引 ````\ds``` 只显示索引 ```\dv```只显示视图

>```\dv``` 只显示视图

## 显示sql执行时间 

> ```\timing on```

## 显示所有schema

> ```\dn``` 

## 显示所有表空间 

> ```\db```
## 显示数据库所有角色和用户
> ```\dg或者\du``` 二者等价

## 显示表的权限分配

> ```` \dp 表名``` 或者 ```\z 表名```

## 指定客户端编码

> ```\encoding utf8```

## 设置查询结果显示 

> ```\pset border 0``` 其中 0为无边框 1为内边框 2为全边框

## 将数据拆分成单元

> ``` \x``` 与mysql的\G 类似

## 执行外部文件的sql

> ```\i 文件名```

## echo 显示

> ```\echo hello world ``` 常用于在执行sql时显示执行注释

## 命令帮助

> ```\?``` 同时也可以使用\d? 查询\d的相关命令

## 自动补全和历史
> 通过上线键可以查看执行历史记录，通过双击tab可以实现补全

## 事务   
> postgresql 自动提交事务，所以可以使用```begin```和```rollback```使事务回滚或者关闭自动提交```\set AUTOCOMMIT off``` 其中AUTOCOMMIT 需要大写 否则会不起作用

## 显示“\”开头的sql语句

>在连接数据库的时候 使用-E 参数可以将\开头的命令对应的sql也打印出来，同时可以使用```\set ECHO_HIDDEN on ```和```\set ECHO_HIDDEN off ```隐藏或者显示命令对应的sql

## 配置参数
> 所有的配置参数都在pg_settings中，其中enumval是取值，unit是单位，short_desc和extra_desc是描述，context表示的是参数的类型

>参数类型，internal:只读参数，程序初始化的时候写死的。postmaster：改变这些参数需要重启postgresql实例。sighup：在postgresql.conf配置的参数，修改这些不需要重启实例，只需要向postmaster发送sighup信号，在master接收到信号之后会向他的子进程发送对应的信号，使新的参数在所有的进程中都生效。backend：在postgresql.conf中，修改这些配置不需要重启服务器，只需要发送sighup信号，但是这些修改在对修改后新建的连接有效，对修改前的连接无效。superuser:只有超级用户在当前的session有效，向postmaster发送sighup也只会影响修改后的连接。user：和superuser唯一的区别就是这个普通用户都能修改

## 访问控制
> postgresql中那些ip可以访问是在pg_hba.conf中配置的

>第一个字段：local、host、hostssl、hostnossl中的提个

>第二个数据库：all代表全部，replication允许流复制链接

>第三个用户名称：all则为所有用户

>第四个为ip：可以访问的ip

>第五个为验证方法：trust、reject、md5、ident、password、gss、sspi、krb5、ldap、radius、cert、pam 但一般只有前几个用的多

>第六个为认证选项

## 取消运行的sql
> pg_cancel_backend(pid)取消一个正在执行的sql

>pg_terminate_backend(pid) cancel只是添加了取消标志，在无法取消时可以使用这个

## explain
> cost=后面的两个值分别为启动成本[返回第一条数据的成本]和返回所有数据的成本

## 系统字段
> oid:行对象标识符

>tableoid: 包含本行的表的iod

>xmin:插入该行版本的事务id

>xmax：删除此行时的事务id，当插入的时候此字段为0，不为0则可能是删除该行的事务还没有提交

>cmin:事务内部插入类操作的命令id

>cmax:事务内部删除类操作的命令id

>ctid：一个行版本在它所处表内的物理位置




 
