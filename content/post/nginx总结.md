---
title: "Nginx总结"
date: 2021-02-19T19:40:57+08:00
categories:
- nginx
- 配置
tags:
- nginx
- 配置
keywords:
- nginx
thumbnailImage: /img/vgj7pl.jpg
autoThumbnailImage: true
thumbnailImagePosition: "left"
coverImage: /img/zm9wxg.jpg
metaAlignment: center
coverMeta: out
---

<!--more-->
# nginx相关的总结--不断完善

## 关于万年长谈的"/"问题

> 在反向代理中【location】和【proxy_pass】的斜杠问题总结

## location中结尾没有/: 

> 即  ```location  /admin``` 此时无论proxy_pass结尾是否存在/ 可以统一记忆 将```/admin```匹配的地址全部追加到 proxy_pass 

> 1、proxy_pass http://ip:port 这种情况访问 /admin/test实际访问的地址为  http://ip:port/admin/test

> 2、proxy_pass http://ip:port/ 这种情况访问 /admin/test 则实际访问地址为 http://ip:port//admin/test 只是变成两个/ 


## location中结尾有/: 
> 即  ```location  /admin/``` 此时有两种情况

> 1、如果proxy_pass结尾存在/ 即: http://ip:port/ 那么此时就将匹配后的内容去掉location的内容加到proxy_pass 例如：访问/admin/test 代理后访问的则是http://ip:port/test

> 2、如果proxy_pass 结尾没有/ 即http://ip:port 那么此时 将location匹配的整个url添加到proxy_pass 后面 例如：访问/admin/test 代理后地址为 http://ip:port/admin/test

## 整体的记忆规则是【当location结尾没有/,此时proxy_pass也没有必要加。当location结尾有斜杠，此时要看你的目的：如果是想原封不动，那就不要加/。如果想去除location的部分，那就在proxy_pass后面加上/】

[nginx配置地址](https://www.cnblogs.com/hmII/p/12731453.html "教程")