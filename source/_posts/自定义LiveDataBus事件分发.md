---
title: 自定义LiveDataBus事件分发
date: 2019-05-17 20:31:36
tags: android
categories: android
comments: false
---
对于Android系统来说，消息传递是最基本的组件，每一个App内的不同页面，不同组件都在进行消息传递。消息传递既可以用于Android四大组件之间的通信，也可用于异步线程和主线程之间的通信。对于Android开发者来说，经常使用的消息传递方式有很多种，从最早使用的Handler、BroadcastReceiver、接口回调，到近几年流行的通信总线类框架EventBus、RxBus。Android消息传递框架，总在不断的演进之中。
<!-- more -->

``` bash
class LiveDataBus private constructor() {

    private val mCaches = HashMap<String, MutableLiveData<Any>>()

    companion object {
        val with by lazy(LazyThreadSafetyMode.SYNCHRONIZED) { LiveDataBus() }
    }

    fun <T> get(owner: LifecycleOwner, target: String, onChange: (T?) -> Unit) {
        if (!mCaches.containsKey(target)) {
            mCaches[target] = MutableLiveData()
        }
        mCaches[target]?.let { liveData ->
            (liveData as MutableLiveData<T>).observe(owner, Observer { onChange(it) })
        }
    }

    fun post(target: String, value: Any) {
        if (mCaches.containsKey(target)) {
            mCaches[target]?.postValue(value)
        }
    }

}
```