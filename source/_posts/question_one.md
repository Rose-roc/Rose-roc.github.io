---
title: question_one
date: 2021-03-02 11:20:27
tags: 
 - android
categories: 刷题
thumbnail: /img/05.png
---



### 今日份刷题

#### onSaveInstanceState方法会在什么时候被执行？

1、当用户按下HOME键时。 这是显而易见的，系统不知道你按下HOME后要运行多少其他的程序，自然也不知道activity A是否会被销毁，故系统会调用onSaveInstanceState，让用户有机会保存某些非永久性的数据。以下几种情况的分析都遵循该原则 2、长按HOME键，选择运行其他的程序时。 3、按下电源按键（关闭屏幕显示）时。 4、从activity A中启动一个新的activity时。 5、屏幕方向切换时，例如从竖屏切换到横屏时。

#### 简述View Touch事件传递机制。

dispatchTouchEvent：进行事件的分发（传递）。返回值是 boolean 类型，受当前onTouchEvent和下级view的dispatchTouchEvent影响

onInterceptTouchEvent：对事件进行拦截。该方法只在ViewGroup中有，View（不包含 ViewGroup）是没有的。一旦拦截，则执行ViewGroup的onTouchEvent，在ViewGroup中处理事件，而不接着分发给View。且只调用一次，所以后面的事件都会交给ViewGroup处理。

onTouchEvent：进行事件处理。

#### 为什么在子线程中执行 new Handler() 会抛出异常?

当创建Handler时会先获取当前线程的Looper，若Looper为null则抛出异常。

#### invalidate()和postInvalidate()的区别？

invalidate()与postInvalidate()都用于刷新View，主要区别是invalidate()在主线程中调用，若在子线程中使用需要配合handler；而postInvalidate()可在子线程中直接调用。

#### res目录和assets目录的区别？

res/raw中的文件会被映射到R.java文件中，访问时可直接使用资源ID，不可以有目录结构。

assets文件夹下的文件不会被映射到R.java中，访问时需要AssetManager类，可以创建子文件夹。

#### onTouch()、onTouchEvent()和onClick()关系？

优先度onTouch()>onTouchEvent()>onClick()。因此onTouchListener的onTouch()方法会先触发；如果onTouch()返回false才会接着触发onTouchEvent()，同样的，内置诸如onClick()事件的实现等等都基于onTouchEvent()；如果onTouch()返回true，这些事件将不会被触发。

#### android中如何处理耗时操作， 有哪几种方法？为什么子线程不能更新UI？

Android处理耗时的操作基本思路为将耗时的操作放到非UI线程执行。常用的是AsyncTask，Handler和Thread，Loaders. 
主要问题是java的线程安全，如果不在主线程更新ui，多个子线程同时给TextView设值，TextView的显示就会出现问题，不知道最终显示哪一个线程的值