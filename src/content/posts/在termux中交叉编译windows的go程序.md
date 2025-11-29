---
title: termux交叉编译windows的go程序
published: 2025-11-29
description: ''
image: ''
tags: [go, 编译]
category: 'go'
draft: false 
lang: ''
---
# 介绍

因为termux源里有`llvm-mingw`,所以在`CGO`（默认开启)开始的情况下，指定`CC`路径为`llvm-mingw`的特定架构的编译器，就可以交叉编译windows程序了。

比如说我想要编译x86_64架构的windows程序:

```bash
GOOS=windows GOARCH=amd64 CC=x86_64-w64-mingw32-clang go build xxx.go
```