---
title: Android MVP架构
date: 2019-05-20 10:22:05
tags: android
categories: mvp
comments: false
---
Model-View-Presenter是一个分离关注点的软件架构。Presenter作为Model和View之间的桥梁，用于演示业务逻辑。
{% asset_img 20160522153041711.jpeg  %}
<!-- more -->
### View
对于View层也是视图层，在View层中只负责对数据的展示，提供友好的界面与用户进行交互。在Android开发中通常将Activity或者Fragment作为View层。
### Presenter
对于Presenter层他是连接View层与Model层的桥梁并对业务逻辑进行处理。在MVP架构中Model与View无法直接进行交互。所以在Presenter层它会从Model层获得所需要的数据，进行一些适当的处理后交由View层进行显示。这样通过Presenter将View与Model进行隔离，使得View和Model之间不存在耦合，同时也将业务逻辑从View中抽离。
### Model
对于Model层也是数据层。它区别于MVC架构中的Model，在这里不仅仅只是数据模型。在MVP架构中Model它负责对数据的存取操作，例如对数据库的读写，网络的数据的请求等。

注意点：

    View不与Model直接交互，而是通过与Presenter交互来与Model间接交互。

    Presenter与View的交互是通过接口来进行的，更有利于添加单元测试

    通常View与Presenter是一对一的，但复杂的View可能绑定多个Presenter来处理逻辑。
{% asset_img 1G020140036-5C1-7.jpeg  %}