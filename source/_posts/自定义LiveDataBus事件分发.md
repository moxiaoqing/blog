---
title: 自定义LiveDataBus事件分发
date: 2019-05-17 20:31:36
tags: android
---
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