---
title: git常用配置
date: 2019-04-01 21:22:05
tags: git
comments: true
---
**保存账号和密码**
``` bash
git config --global credential.helper store
```
**忽略ssl验证**
``` bash
git config --global http.sslVerify "false"
``` 