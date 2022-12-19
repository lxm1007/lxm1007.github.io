---
title: "Rust学习笔记"
date: 2022-06-28T11:19:24+08:00
categories:
- rust
- 学习笔记
tags:
- rust
- 学习笔记
keywords:
- rust
thumbnailImage: /img/Hippopx-01.webp
autoThumbnailImage: true
thumbnailImagePosition: "left"
coverImage: /img/Hippopx-02.webp
metaAlignment: center
coverMeta: out
---

<!--more-->


## macos 安装rust环境

### 1、```brew install rustup```
### 2、``` rustup-init ```
### 3、```source $HOME/.cargo/env```
### 4、检查安装是否成功 ```rustc --version``` ```cargo --version```
### 5、检查安装的rust版本列表 ```rustup toolchain list ```
### 6、安装其他版本rust ```rustup install 1.59```
### 7、再次执行```rustup toolchain list ``` 查看安装列表
### 8、```rustup default 1.59``` 切换默认的rust版本
### 9、```rustup component add rust-src ``` 解决vscode下的报错问题

## idea 安装rust 插件 

## ```rustc main.rs``` 编译rust文件

## ```./main ``` 运行二进制文件

## cargo的使用

### ``` cargo new hello_cargo```  创建项目 cargo 版本依赖管理
### ```cargo build ``` 构建二进制文件
### ```cargo run  ``` 构建并运行
### ```cargo check``` 检查是否可编译 但是不运行
### ``` cargo build --release ``` 发布正式版本，区别于debug版本，耗时长但是性能更好 运行更快
### ```cargo doc --open``` 生成文档并在本地打开

## Cargo.toml配置文件
### ```rand = "0.3.14"``` 添加依赖 0.3.14 事实上是 ^0.3.14 的简写，它表示任何与 0.3.14 版本公有 API 相兼容的版本 


## 语法
### ```loop``` 循环使用 ``` break``` 跳出循环
### ``` match {}``` 

### ``` i8 u8 i16 u16 i32 u32 i64 u64 ```

### 在debug模式下 整形溢出会panic 但是在release下 不检测溢出 256变成0 257变成1 依此类推

### 元组解构 ``` let tup = (500, 6.4, 1); let (x, y, z) = tup; ```




