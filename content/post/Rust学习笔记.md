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
