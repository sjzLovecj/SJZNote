---
title: 'iOS安装包瘦身'
date: 2020-08-27 10:00:41
tags: []
published: true
hideInList: false
feature: 
isTop: false
---
安装包（IPA）主要由可执行文件、资源组成

- 资源（图片、音频、视频等）
    - 采用无损压缩
    - 去除没有用到的资源（LSUnusedResources）

- 可执行文件瘦身
    - 编译器优化
        - Strip Linked Product、Make Strings Read-Only、Symbols Hidden by Default 设置为YES
        - 去掉异常支持，Enable C++ Exceptions、Enable Objective-C Exceptions设置为NO，Other C Flags添加-fno-exceptions
        - 利用AppCode 检测未使用的代码：菜单栏->Code->Inspect Code
        - 编写LLVM插件检测出重复代码、未被调用的代码

- LinkMap
    - 生成LinkMap文件，可以查看可执行文件的具体组成