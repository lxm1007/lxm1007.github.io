---
title: "Curl命令"
date: 2021-02-20T10:10:53+08:00
categories:
- sell
- curl
tags:
- shell
- curl
keywords:
- curl
thumbnailImage: /img/dgkq7g.webp
autoThumbnailImage: true
thumbnailImagePosition: "left"
coverImage: /img/70pomn.jpg
metaAlignment: center
coverMeta: in
---

<!--more-->
# curl命令总结
[参考教程](http://www.ruanyifeng.com/blog/2019/09/curl-reference.html "curl")

## 【-A】 User-Agent 也可以通过设置请求头 ```-H 'User-Agent: php/1.0' ```指定

## 【-b】设置cookie ```-b 'foo=bar'```

## 【-c】将服务器设置的cookie写入文件 ```curl -c 1.txt www.baidu.com```

## 【-d|--data-urlencode】常用：post请求提交body，HTTP 请求会自动加上标头Content-Type : application/x-www-form-urlencoded，所以可以省略 ```-X POST```。--data-urlencode 会自动将发送的数据进行url编码

>```curl -d '{"login": "emma", "pass": "123"}' -H 'Content-Type: application/json' www.baidu.com``` 提交json数据

> ```curl -d'login=emma＆password=123' https://google.com/login ``` 提交urlencode数据

## 【-e】 设置HTTP 的标头Referer，可以使用``` -H 'Referer: https://google.com?q=example'``` 

## 【-F】上传文件，HTTP 请求加上标头Content-Type: multipart/form-data，默认上传的字段为file
> ```curl -F 'file=@photo.png;type=image/png'```上传photo图片，设置MIME类型为png，如果不设置默认为application/octet-stream

> ```curl -F 'file=@photo.png;filename=me.png'``` 可以指定文件名,如果不指定默认为上传的文件名
## 【-G】 用来构造 URL 的查询字符串
>```curl -G -d 'wd=java' http://www.baidu.com/s```
## 【-H】设置请求头
## 【-i】打印响应的标头
## 【-I | --head】 发送head请求 几乎不用
## 【-k】常用在自签证书 跳过 SSL 检测
## 【-L】常用 让 HTTP 请求跟随服务器的重定向。curl 默认不跟随重定向
## 【--limit-rate】 模拟慢网速情况。
> ```curl --limit-rate 200k https://google.com```
## 【-o】 等同wget -o 用于保存服务响应
>  ```curl -o baidu.html baidu.com```
## [-O] 将服务器回应保存成文件，并将 URL 的最后部分当作文件名
> ``` curl -O https://www.example.com/foo/bar.html``` 则文件名为bar.html
## 【-u】用来设置服务器认证的用户名和密码，在url上指定更常用
> ```curl -u 'bob:12345' https://google.com/login```

>``` curl https://bob:12345@google.com/login```

## 【-v】 输出通信的整个过程，用于调试
> ```curl -v baidu.com```

## 【-x】 使用代理协议
>```curl -x socks5://james:cats@myproxy.com:8080 https://www.example.com``` 使用socks5协议

## 【-X】 指定请求方式