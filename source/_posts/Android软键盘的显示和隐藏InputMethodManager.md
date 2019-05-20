---
title: Android软键盘的显示和隐藏InputMethodManager
date: 2019-05-20 09:40:33
tags: android
categories: android
comments: false
---
如果有需要用到输入的地方，通常会有需要自动弹出或者收起软键盘的需求。想要操作软键盘，需要使用到 InputMethodManager ，它是一个系统服务，可以使用 Context.getSystemService() 获取到它。而很多关键的逻辑代码，都是在 InputMethodManagerService 中实现的。
<!-- more -->
### 显示软键盘
在 InputMethodManager 中，有两个方法 showSoftInput() 和 showSoftInputFromInputMethod() ，而实际上，只有 showSoftInput() 是有效的。它有两个重载方法，而通常我们会使用它的两个参数的方法。
``` bash
/**
 * Synonym for {@link #showSoftInput(View, int, ResultReceiver)} without
 * a result receiver: explicitly request that the current input method's
 * soft input area be shown to the user, if needed.
 *
 * @param view The currently focused view, which would like to receive
 * soft keyboard input.
 * @param flags Provides additional operating flags.  Currently may be
 * 0 or have the {@link #SHOW_IMPLICIT} bit set.
 */
public boolean showSoftInput(View view, int flags) {
    return showSoftInput(view, flags, null);
}
```
这里我们只需要传递两个参数。它首先需要一个 View ，使用软键盘就是为了输入，而输入就需要有接收输入内容的 View ，这里接收输入的 View ，最好是一个 EditText（但这不是必须的）。
而第二个参数 flags 就是个标志位，直接传0即可。
在 onCreate() 中，如果立即调用 showSoftInput() 是不会生效的。想要在页面一启动的时候就弹出键盘，可以在 Activity 上，设置 android:windowSoftInputMode 属性来完成，或者做一个延迟加载，View.postDelayed() 也是一个解决方案。
### 隐藏软键盘
虽然 showSoftInput() 方法是有效的，但是想要隐藏软键盘，就没有提供对应的 hideSoftInput() 方法，但是却有一个 hideSoftInputFromWindow() 方法，可以用来隐藏软键盘。
``` bash
/**
 * Synonym for {@link #hideSoftInputFromWindow(IBinder, int, ResultReceiver)}
 * without a result: request to hide the soft input window from the
 * context of the window that is currently accepting input.
 *
 * @param windowToken The token of the window that is making the request,
 * as returned by {@link View#getWindowToken() View.getWindowToken()}.
 * @param flags Provides additional operating flags.  Currently may be
 * 0 or have the {@link #HIDE_IMPLICIT_ONLY} bit set.
 */
public boolean hideSoftInputFromWindow(IBinder windowToken, int flags) {
    return hideSoftInputFromWindow(windowToken, flags, null);
}
```
它接收两个参数，第一个参数是一个 IBinder ，可以直接传递一个 View.getWindowToken() 的 windowToken 对象就可以了。而第二个参数，就是隐藏软键盘的标志位，如果没有特殊要求的话，直接传递 0 就好了。
注意这里虽然原则上需要传递一个之前弹出键盘传递的时候，传递的 View 的 windowToken ，但是实际情况是你只需要传递一个存在于当前布局 ViewTree 中，随意一个 View 的 windowToken 就可以了。
### 切换键盘的弹出和隐藏
在 InputMethodManager 中，还提供了一个 toggleSoftInput() 方法，如同它的名字一样，它可以让软键盘在显示和隐藏之间切换。
``` bash
/**
 * This method toggles the input method window display.
 *
 * If the input window is already displayed, it gets hidden.
 * If not the input window will be displayed.
 * @param showFlags Provides additional operating flags.  May be
 * 0 or have the {@link #SHOW_IMPLICIT},
 * {@link #SHOW_FORCED} bit set.
 * @param hideFlags Provides additional operating flags.  May be
 * 0 or have the {@link #HIDE_IMPLICIT_ONLY},
 * {@link #HIDE_NOT_ALWAYS} bit set.
 */
public void toggleSoftInput(int showFlags, int hideFlags) {
    if (mCurMethod != null) {
        try {
            mCurMethod.toggleSoftInput(showFlags, hideFlags);
        } catch (RemoteException e) {
        }
    }
}
```
该方法，接收两个 flags ，分别是控制 show 和 hide 时候的标识，它们的含义和前面介绍的 showSoftInput() 和 hideSoftInputFromWindow() 一致，所以没有特殊要求，直接传递 0 就好了。
toggleSoftInput() 方法不要求传递一个 View 或者 windowToken，所以它并没有 showSoftInput() 中的一些限制，但是依然还有需要在布局绘制完成之后调用才会有效果。
虽然这个方法，限制很少，但是我们基本上不会使用它。主要原因在于，它是一个开关的方法，会根据当前的状态做相反的操作。这就导致很多时候，我们在代码中，无法直接根据 InputMethodManager 提供的方法判断当前软键盘的显示状态，这样也就无法确定调用它的时候的效果了。






