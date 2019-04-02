---
title: jdk的安装与环境配置
date: 2019-04-01 20:56:35
tags: java
comments: true
---
https://www.oracle.com/technetwork/java/javase/downloads/index.html

### Mac
1.打开终端Terminal
2.vim .bash_profile
3.输入如下配置,然后wq保存关闭该窗口
``` bash
JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_191.jdk/Contents/Home
PATH=$JAVA_HOME/bin:$PATH:.
CLASSPATH=$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:.
export JAVA_HOME
export PATH
export CLASSPATH
```
4.使用"source .bash_profile"使配置生效

### Ubuntu
1.打开终端Terminal
2.sudo vim /etc/profile
3.输入如下配置,然后wq保存关闭该窗口
``` bash
export JAVA_HOME=/usr/lib/jdk/jdk1.8.0_191
export JRE_HOME=${JAVA_HOME}/jre    
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib    
export PATH=${JAVA_HOME}/bin:$PATH 
```
4.使用"source /etc/profile"使配置生效

### Window
1.新建系统变量：JAVA_HOME=C:\Program Files\Java\jdk1.8.0_191
2.在系统变量Path后面追加;%JAVA_HOME%\bin
3.新建系统变量：CLASSPATH=.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar