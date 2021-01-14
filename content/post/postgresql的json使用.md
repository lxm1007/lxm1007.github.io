---
title: "Postgresql的json使用"
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
coverImage: /img/2020.jpg
metaAlignment: center
coverMeta: out
---

<!--more-->
因为一直很好奇，为什么网上说postgresql的速度比mysql要快很多，但是国内用的人却很少，况且，postgresql还支持json。
于是抱着试试看的态度用docker部署一个postgresql作测试。记录过程如下

* 部署postgresql `docker run -d -p 5432:5432  -v /my/own/datadir:/var/lib/postgresql/data -e POSTGRES_PASSWORD=123456 ` 因为不想持久化数据 所有就不挂载数据了
* 修改远程登录 `docker exec -it 732042e356e8 psql -U postgres -d postgres -h 127.0.0.1 -p 5432 `
* 创建数据表test_json 只需要一列数据 json_text数据格式为jsonb (关于json和jsonb的大概区别就是：jsonb在存的时候进行数据处理，包括重复键的覆盖，所以存慢取快，json保留存的时候的重复键，所以存快取慢，大部分应该都是使用存慢取快这种吧)
* 在对应的字段上加上索引`create index idx_json on test_json using gin (json_text)`
* `INSERT INTO test_json(json_text) VALUES('{"name":"lxm","age":12}');` 先插入一条数据 然后试用批量复制的方法创建200W 条数据 `INSERT INTO test_json(json_text) SELECT json_text from test_json`
* 查询 存在key的数据 `SELECT json_text->'name' as name,json_text->'age'as age from test_json  WHERE json_text ? 'name';` 查询json中存在name的key的数据 200w数据大概需要用时7s
* 查询value存在的数据 `SELECT json_text->'name' as name,json_text->'age'as age from test_json  WHERE json_text @> '{"age":12}';` 查询时间6.7s

