---
title: shared-preference
date: 2021-03-02 16:38:37
tags:
 - android
categories: 知识点
thumbnail: /img/07.png
---

### SharedPreference使用

SharedPreferences是Android平台上一个轻量级的存储类，用来保存应用的一些常用配置，比如Activity状态，Activity暂停时，将此activity的状态保存到SharedPereferences中；当Activity重载，系统回调方法onSaveInstanceState时，再从SharedPreferences中将值取出。
 其中的原理是通过Android系统生成一个xml文件保到：/data/data/包名/shared_prefs目录下，类似键值对的方式来存储数据。
 Sharedpreferences提供了常规的数据类型保存接口比如：int、long、boolean、String、Float、Set和Map这些数据类型。

SharedPreference存储形式为键值对形式，下面为存储和获取的代码示例：

#### 存储示例

```java
/**
参数有两个，第一个表示Share文件的名称，不同的名称对应这不同的Share文件，其中的内容也是不同；
第二个参数表示操作模式，操作模式有两种：MODE_PRIVATE和MODE_MULTI_PRIVATE
MODE_PRIVATE：默认操作模式，直接在把第二个参数写0就是默认使用这种操作模式，
这种模式表示只有当前的应用程序才可以对当前这个SharedPreferences文件进行读写。
MODE_MULTI_PRIVATE：用于多个进程共同操作一个SharedPreferences文件。
*/
SharedPreferences sp = context.getSharedPreferences(PREFERENCES_NAME,Context.MODE_PRIVATE);
//获取Editor对象，这个对象用于写入，可理解为编辑
SharedPreferences.Editor editor = sp.edit();
//Editor对象有几个方法需要注：clear()，commit()，putXXX(),clear()为清空Share文件中的内容，
//commit()为提交，editor在put值以后，需要调用commit方法才能被真正写入到Share文件中
editor.putString("uid", "22222").commit();
```

#### 读取示例

```java
//先获取对应的Share
SharedPreferences sp=context.getSharedPreferences(PREFERENCES_NAME,Context.MODE_PRIVATE);
//根据key取出对应的值
sp.getString("uid", "");//第二个参数为默认值，即当从Share中取不到时，返回这个值
```

我们可以使用Share存储一些较轻量的信息，比如我们可以使用Share存储一个值，使用这个值可以判断APP是不是第一次打开。

#### 要点

1. 上面用到的是Context类的getSharedPreferences()方法，需要传入文件名和操作模式，默认为0也就是MODE_PRIVATE。
获取SharedPreferences还有两种方法：Activity类的getPreferences()方法，和PreferenceManager类的静态方法getDefaultSharedPreferences()。前者会自动将当前类名作为文件名，只需要传入操作模式。后者需传入context，并自动使用包名作为前缀来命名SharedPreferences文件。

2. 提交SharedPreferences数据时，可以用SharedPreferences.Editor的commit()方法，也 可以用它的apply()方法。两者有什么区别呢，下面的解释来自《阿里巴巴Android开发手册》：

  SharedPreference 提 交 数 据 时 ， 尽 量 使 用 Editor#apply()，而非Editor#commit()。一般来讲，仅当需要确定提交结果，并据此有后续操作时，才使用 Editor#commit()。

  说明：

  SharedPreference 相关修改使用 apply 方法进行提交会先写入内存，然后异步写入磁盘，commit
  方法是直接写入磁盘。如果频繁操作的话 apply 的性能会优于 commit，apply会将最后修改内容写入磁盘。但是如果希望立刻获取存储操作的结果，并据此做相应的其他操作，应当使用 commit。