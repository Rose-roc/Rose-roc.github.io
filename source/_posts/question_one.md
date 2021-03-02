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

#### 请写出下面代码输出结果是怎么样的。

public class MainThreadTestActivity extends AppCompatActivity {

  private static final String TAG = MainThreadTestActivity.class.getSimpleName();

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main_thread_test);

```java
View view = new View(this);
view.post(new Runnable() {
  @Override
  public void run() {
    Log.i(TAG, "[view.post] >>>> 1 ");
  }
});

new Handler(Looper.getMainLooper()).post(new Runnable() {
  @Override
  public void run() {
    Log.i(TAG, "[handler.post] >>>> 2");
  }
});

runOnUiThread(new Runnable() {
  @Override
  public void run() {
    Log.i(TAG, "[runOnUiThread] >>>>> 3");
  }
});

new Thread(new Runnable() {
  @Override
  public void run() {
    runOnUiThread(new Runnable() {
      @Override
      public void run() {
        Log.i(TAG, "[runOnUiThread from thread] >>>> 4");
      }
    });
  }
}).start();
```
参考答案

[runOnUiThread] >>>>> 3

[handler.post] >>>> 2 

[runOnUiThread from thread] >>>> 4 

[view.post] >>>> 1

![](\question_one\1604d24577c23f9f.png)

如果当前不是 UI 线程，那么由主线程的 Handler 扔个消息给 MessageQueue；如果当前是 UI 线程，则立刻执行。

#### 软引用和弱引用的区别？

软引用关联的对象只有在内存不足时才会被回收，而被弱引用关联的对象在JVM进行垃圾回收时总会被回收。

#### SharedPreference的apply和commit的区别?

1、apply没有返回值，commit会返回一个Boolean值，表明是否修改成功

2、apply是将修改的数据提交到了内存，而后异步真正提交到硬件磁盘；而commit是同步提交到硬件磁盘，因此在多个并发的提交commit的时候，它们会等待正在处理的commit保存到磁盘后再操作，从而降低了效率；而apply只是原子的提交到内容，后面有调用apply的函数，将会直接覆盖前面的内存数据，这样从一定程度上提高了很多效率

3、apply方法不会提示任何失败的提示

由于在一个进程中，sharedPreference是单实例，一般不会出现并发冲突，如果对提交的结果不关心的话，建议使用apply，当然需要确保提交成功且有后续操作的话，还是需要commit的。

#### ScrollView下嵌套一个ListView通常会出现什么问题？如何解决？

①ListView的高度不能完全展开

这种情况是当ScrollView嵌套ListView时，ListView的高度设置为wrap_content时会产生，一般情况下ListView只显示的第一个Item。

正常情况下，高度设置为“wrap_content”的ListView在测量自己的高度会使用MeasureSpec.AT_MOST这个模式高度来返回可包含住其内容的高度。

而实际上当ListView被ScrollView嵌套时，ListView使用的测量模式是ScrollView传入的MeasureSpec.UNSPECIFIED。

**解决方法一**：覆写ListView的onMeasure方法：

```java
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int newHeightMeasureSpec = MeasureSpec.makeMeasureSpec(Integer.MAX_VALUE>>2, // 设计一个较大的值和AT_MOST模式
                MeasureSpec.AT_MOST);
        super.onMeasure(widthMeasureSpec, newHeightMeasureSpec);//再调用原方法测量
    }
```

**解决方法二**：在Activity中动态修改ListView的高度，注意当ListView的子item需要根布局是LinearLayout，或需要为一个View，因为有些布局中没有measure这个方法，比如RelativeLayout。

```java
public void setListViewHeightBasedOnChildren(ListView listView) {
    ListAdapter listAdapter = listView.getAdapter();
    if (listAdapter == null) {
        return;
    }

    int totalHeight = 0;
    for (int i = 0; i < listAdapter.getCount(); i++) {
        View listItem = listAdapter.getView(i, null, listView);
        listItem.measure(0, 0);  // 获取item高度
        totalHeight += listItem.getMeasuredHeight();
    }

    ViewGroup.LayoutParams params = listView.getLayoutParams();
    // 最后再加上分割线的高度和padding高度，否则显示不完整。
    params.height = totalHeight + (listView.getDividerHeight() * (listAdapter.getCount() - 1))+listView.getPaddingTop()+listView.getPaddingBottom();
    listView.setLayoutParams(params);
}
```

**缺陷**：上述两种解决方法都有一个缺陷就是，当第一次进入界面动态加载listview的i数据后，ScrollView会跳滑到listview的第一个子项。处理缺陷可以有两种方式：

方式一：
先设置ListView失去焦点，这并不影响item的点击事件发生

```java
XXX.setFocusable(false);
```

方式二：
重新调整ScrollView的位置

```java
mScrollView.post(new Runnable() {
        @Override
        public void run() {
            mScrollView.scrollTo(0,0); 
        }
  });
```

② ListView高度固定后无法触发滑动时间

当对listview的高度设置为固定值（例200dp）时，listview的高度是可以直接显示出来的。但嵌套在一起后ScrollView中的ListView就没法上下滑动了，事件先被ScrollView响应了。

解决方法：当ListView自身接收到的滑动事件时，使ScrollView取消拦截。ListView区域内的滑动事件由自己处理，ListView区域外滑动事件由外层ScrollView处理。可以系统自带的API来实现：requestDisallowInterceptTouchEvent这一方法。
**解决方法一**：在这里我们自定义ListView来重写ListView的dispatchTouchEvent函数：

```java
	@Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
            case MotionEvent.ACTION_MOVE:
                getParent().requestDisallowInterceptTouchEvent(true);
                break;
            case MotionEvent.ACTION_UP:
            case MotionEvent.ACTION_CANCEL:
                getParent().requestDisallowInterceptTouchEvent(false);
                break;
        }
        return super.dispatchTouchEvent(ev);
    }
```

**解决方式二**：或者也可以给ListView绑定触摸事件的监听：

```java
scrollListView.setOnTouchListener(new View.OnTouchListener() {
    @Override
    public boolean onTouch(View v, MotionEvent ev) {
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
            case MotionEvent.ACTION_MOVE:
                scrollListView.getParent().requestDisallowInterceptTouchEvent(true);
                break;
            case MotionEvent.ACTION_UP:
            case MotionEvent.ACTION_CANCEL:
                scrollListView.getParent().requestDisallowInterceptTouchEvent(false);
                break;
        }
        return false;
    }
});
```
#### 启动一个SingleTop模式的Activity，然后再次启动一次它，它的生命周期如何变化呢？

onCreate -> onStart -> onResume -> onPause -> onNewIntent -> onResume。

#### FragmentPagerAdapter和FragmentStatePagerAdapter区别？

在于fragment 存储、恢复、销毁 的方式不同，当Viewpager中fragment数量多的时候用FragmentStatePagerAdapter，反之则用FragmentPagerAdapter。

#### 如何开启一个新的进程？Application在多进程下会多次调用onCreate() 吗？

当采用多进程的时候，比如下面的Service 配置：

```xml
<service
    android:name=".MyService"
    android:enabled="true"
    android:exported="false"
    android:process=":remote" />
```

android:process 属性中 :的作用就是把这个名字附加到你的包所运行的标准进程名字的后面作为新的进程名称。

这样配置会调用 onCreate() 两次。

#### 什么是OOM？检测OOM的机制是什么？如何避免？

内存泄漏。

WeakReference与ReferenceQueue联合使用，在弱引用关联的对象被回收后，会将引用添加到ReferenceQueue；清空后，可以根据是否继续含有该引用来判定是否被回收；判定回收， 手动GC, 再次判定回收，采用双重判定来确保当前引用是否被回收的状态正确性；如果两次都未回收，则确定为泄漏对象。


如何避免内存泄露：
1.使用缓存技术，比如LruCache、DiskLruCache、对象重复并且频繁调用可以考虑对象池
2.对于引用生命周期不一样的对象，可以用软引用或弱引用SoftReferner WeakReferner
3.对于资源对象 使用finally 强制关闭
4.内存压力过大就要统一的管理内存

#### 请写出广播的两种注册形式。他们区别在哪？

第一种：使用代码进行订阅

```java
IntentFilter filter = new IntentFilter("android.provider.Telephony.SMS_RECEIVED"); 
IncomingSMSReceiver receiver = new IncomingSMSReceiver(); 
registerReceiver(receiver, filter); 
```



第二种：在AndroidManifest.xml文件中的节点里进行订阅:

```xml
<receiver android:name=".IncomingSMSReceiver"> 

 	<intent-filter> 

 		<action android:name="android.provider.Telephony.SMS_RECEIVED"/> 

	</intent-filter> 

</receiver> 
```

在AndroidManifest中进行注册后，不管改应用程序是否处于活动状态，都会进行监听。

在代码中进行注册后，当应用程序关闭后，就不再进行监听。

#### Activity A 跳转Activity B，Activity B再按back键回退，两个过程各自的生命周期

ActivityA和ActivityB生命周期执行顺序如下： A.onPause －> B.onCreate －> B.onStart－> B.onResume－> A.onStop

ActivityB 按back键回退
按下back键后： B.onPause－>A.onRestart－>A.onStart－>A.onResume－>B.onStop－>B.onDestory

#### 聊聊RecyclerView的缓存机制。

RecyclerView是四级缓存。

一级缓存  mAttachedScrap和mChangedScrap  这是优先级最高的缓存，RecyclerView在获取ViewHolder时,优先会到这两个缓存来找。其中mAttachedScrap存储的是当前还在屏幕中的ViewHolder，mChangedScrap存储的是数据被更新的ViewHolder,比如说调用了Adapter的notifyItemChanged方法。可能有人对这两个缓存还是有点疑惑，不要急，待会会详细的解释。
二级缓存  mCachedViews  默认大小为2，通常用来存储预取的ViewHolder，同时在回收ViewHolder时，也会可能存储一部分的ViewHolder，这部分的ViewHolder通常来说，意义跟一级缓存差不多。
三级缓存  ViewCacheExtension  自定义缓存,通常用不到，在本文中先忽略
四级缓存  RecyclerViewPool  根据ViewType来缓存ViewHolder，每个ViewType的数组大小为5，可以动态的改变。