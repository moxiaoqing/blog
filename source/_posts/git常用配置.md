---
title: git常用配置
date: 2019-04-01 21:22:05
tags: git
categories: git
comments: false
---
Git是一个开源的分布式版本控制系统，可以有效、高速地处理从很小到非常大的项目版本管理。
<!-- more -->
**保存账号和密码**
``` bash
git config --global credential.helper store
```
**忽略ssl验证**
``` bash
git config --global http.sslVerify "false"
``` 