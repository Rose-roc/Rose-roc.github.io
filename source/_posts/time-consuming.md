---
title: 处理耗时操作的几种方法
date: 2021-03-02 12:53:43
tags:
 - android
categories: 刷题
thumbnail: /img/06.png
---

### 处理耗时操作的几种方法

#### ① AsyncTask

##### 1. 定义

- 一个android已封装好的轻量级异步类
- 抽象类

```java
public abstract class AsyncTask<Params, Progress, Result> { 
 ... 
 }
```

##### 2. 作用

- 实现多线程，在工作线程中执行任务，如耗时任务

- 异步通信、信息传递

  实现工作线程与主线程（UI线程）之间的通信，保证线程安全

##### 3. 优点

- 方便实现异步通信
  不需使用 “任务线程（如继承`Thread`类） + `Handler`”的复杂组合
- 节省资源
  采用线程池的缓存线程 + 复用线程，避免了频繁创建 & 销毁线程所带来的系统资源开销

##### 4. 使用

```java
public abstract class AsyncTask<Params, Progress, Result> { 
 ... 
}

// 类中参数为3种泛型类型
// 整体作用：控制AsyncTask子类执行线程任务时各个阶段的返回类型
// 具体说明：
    // a. Params：开始异步任务执行时传入的参数类型，对应excute（）中传递的参数
    // b. Progress：异步任务执行过程中，返回下载进度值的类型
    // c. Result：异步任务执行完成后，返回的结果类型，与doInBackground()的返回值类型保持一致
// 注：
    // a. 使用时并不是所有类型都被使用
    // b. 若无被使用，可用java.lang.Void类型代替
    // c. 若有不同业务，需额外再写1个AsyncTask的子类
}
```

###### 核心方法

![核心方法](\time-consuming\944365-153fb37764704129.webp)

###### 执行顺序

![执行顺序](\time-consuming\944365-31df794006c69621.webp)

###### 使用步骤

1. 创建 `AsyncTask` 子类 & 根据需求实现核心方法
2. 创建 `AsyncTask`子类的实例对象（即 任务实例）
3. 手动调用`execute()`从而执行异步线程任务

```java
/**
  * 步骤1：创建AsyncTask子类
  * 注： 
  *   a. 继承AsyncTask类
  *   b. 为3个泛型参数指定类型；若不使用，可用java.lang.Void类型代替
  *   c. 根据需求，在AsyncTask子类内实现核心方法
  */

  private class MyTask extends AsyncTask<Params, Progress, Result> {

        ....

      // 方法1：onPreExecute（）
      // 作用：执行 线程任务前的操作
      // 注：根据需求复写
      @Override
      protected void onPreExecute() {
           ...
        }

      // 方法2：doInBackground（）
      // 作用：接收输入参数、执行任务中的耗时操作、返回 线程任务执行的结果
      // 注：必须复写，从而自定义线程任务
      @Override
      protected String doInBackground(String... params) {

            ...// 自定义的线程任务

            // 可调用publishProgress（）显示进度, 之后将执行onProgressUpdate（）
             publishProgress(count);
              
         }

      // 方法3：onProgressUpdate（）
      // 作用：在主线程 显示线程任务执行的进度
      // 注：根据需求复写
      @Override
      protected void onProgressUpdate(Integer... progresses) {
            ...

        }

      // 方法4：onPostExecute（）
      // 作用：接收线程任务执行结果、将执行结果显示到UI组件
      // 注：必须复写，从而自定义UI操作
      @Override
      protected void onPostExecute(String result) {

         ...// UI操作

        }

      // 方法5：onCancelled()
      // 作用：将异步任务设置为：取消状态
      @Override
        protected void onCancelled() {
        ...
        }
  }

/**
  * 步骤2：创建AsyncTask子类的实例对象（即 任务实例）
  * 注：AsyncTask子类的实例必须在UI线程中创建
  */
  MyTask mTask = new MyTask();

/**
  * 步骤3：手动调用execute(Params... params) 从而执行异步线程任务
  * 注：
  *    a. 必须在UI线程中调用
  *    b. 同一个AsyncTask实例对象只能执行1次，若执行第2次将会抛出异常
  *    c. 执行任务中，系统会自动调用AsyncTask的一系列方法：onPreExecute() 、doInBackground()、onProgressUpdate() 、onPostExecute() 
  *    d. 不能手动调用上述方法
  */
  mTask.execute()；
```

###### 示例代码

```java
public class MainActivity extends AppCompatActivity {

    // 线程变量
    MyTask mTask;

    // 主布局中的UI组件
    Button button,cancel; // 加载、取消按钮
    TextView text; // 更新的UI组件
    ProgressBar progressBar; // 进度条
    
    /**
     * 步骤1：创建AsyncTask子类
     * 注：
     *   a. 继承AsyncTask类
     *   b. 为3个泛型参数指定类型；若不使用，可用java.lang.Void类型代替
     *      此处指定为：输入参数 = String类型、执行进度 = Integer类型、执行结果 = String类型
     *   c. 根据需求，在AsyncTask子类内实现核心方法
     */
    private class MyTask extends AsyncTask<String, Integer, String> {

        // 方法1：onPreExecute（）
        // 作用：执行 线程任务前的操作
        @Override
        protected void onPreExecute() {
            text.setText("加载中");
            // 执行前显示提示
        }


        // 方法2：doInBackground（）
        // 作用：接收输入参数、执行任务中的耗时操作、返回 线程任务执行的结果
        // 此处通过计算从而模拟“加载进度”的情况
        @Override
        protected String doInBackground(String... params) {

            try {
                int count = 0;
                int length = 1;
                while (count<99) {

                    count += length;
                    // 可调用publishProgress（）显示进度, 之后将执行onProgressUpdate（）
                    publishProgress(count);
                    // 模拟耗时任务
                    Thread.sleep(50);
                }
            }catch (InterruptedException e) {
                e.printStackTrace();
            }

            return null;
        }

        // 方法3：onProgressUpdate（）
        // 作用：在主线程 显示线程任务执行的进度
        @Override
        protected void onProgressUpdate(Integer... progresses) {

            progressBar.setProgress(progresses[0]);
            text.setText("loading..." + progresses[0] + "%");

        }

        // 方法4：onPostExecute（）
        // 作用：接收线程任务执行结果、将执行结果显示到UI组件
        @Override
        protected void onPostExecute(String result) {
            // 执行完毕后，则更新UI
            text.setText("加载完毕");
        }

        // 方法5：onCancelled()
        // 作用：将异步任务设置为：取消状态
        @Override
        protected void onCancelled() {

            text.setText("已取消");
            progressBar.setProgress(0);

        }
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // 绑定UI组件
        setContentView(R.layout.activity_main);

        button = (Button) findViewById(R.id.button);
        cancel = (Button) findViewById(R.id.cancel);
        text = (TextView) findViewById(R.id.text);
        progressBar = (ProgressBar) findViewById(R.id.progress_bar);

        /**
         * 步骤2：创建AsyncTask子类的实例对象（即 任务实例）
         * 注：AsyncTask子类的实例必须在UI线程中创建
         */
        mTask = new MyTask();

        // 加载按钮按按下时，则启动AsyncTask
        // 任务完成后更新TextView的文本
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                /**
                 * 步骤3：手动调用execute(Params... params) 从而执行异步线程任务
                 * 注：
                 *    a. 必须在UI线程中调用
                 *    b. 同一个AsyncTask实例对象只能执行1次，若执行第2次将会抛出异常
                 *    c. 执行任务中，系统会自动调用AsyncTask的一系列方法：onPreExecute() 、doInBackground()、onProgressUpdate() 、onPostExecute()
                 *    d. 不能手动调用上述方法
                 */
                mTask.execute();
            }
        });

        cancel = (Button) findViewById(R.id.cancel);
        cancel.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // 取消一个正在执行的任务,onCancelled方法将会被调用
                mTask.cancel(true);
            }
        });

    }

}
```

![示例图片](\time-consuming\944365-23bdf9a3bc62e825.webp)

#### ② Handle

##### 1. 作用

传递信息Message、子线程通知主线程更新UI

##### 2. 常用api

```java
//消息
Message message = Message.obtain();
//发送消息
    new Handler().sendMessage(message);
//延时1s发送消息
    new Handler().sendMessageDelayed(message, 1000);
//发送带标记的消息(内部创建了message,并设置msg.what = 0x1)
    new Handler().sendEmptyMessage(0x1);
//延时1s发送带标记的消息
    new Handler().sendEmptyMessageDelayed(0x1, 1000);
//延时1秒发送消息（第二个参数为：相对系统开机时间的绝对时间，而SystemClock.uptimeMillis()是当前开机时间）
    new Handler().sendMessageAtTime(message, SystemClock.uptimeMillis() + 1000);
 
//避免内存泄露的方法：
//移除标记为0x1的消息
    new Handler().removeMessages(0x1);
//移除回调的消息
    new Handler().removeCallbacks(Runnable);
//移除回调和所有message
    new Handler().removeCallbacksAndMessages(null);
```
##### 3. 简单使用

主线程中：首先是创建一个Handler对象，并重写handleMessage方法，然后需要消息通信的地方，通过Handler的sendMessage方法发送消息。

```java
private Handler mHandler;

..onCreate(..){
    mHandler = new Handler() {
            @Override
            public void handleMessage(Message msg) {
                super.handleMessage(msg);
                // 处理消息
                }
            }
        };

 	btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // 创建一个子线程，在子线程中发送消息
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        Message msg = Message.obtain();
                        msg.what = MSG_SUB_TO_MAIN;
                        msg.obj = "这是一个来自子线程的消息";
                        // 2.发送消息
                        mHandler.sendMessage(msg);
                    }
                }).start();
            }
        });
}
```

子线程中：

1. 调用Looper.prepare()
2. 创建Handler对象
3. 调用Looper.loop()

```java
// 创建一个子线程，并在子线程中创建一个Handler，且重写handleMessage
        new Thread(new Runnable() {
            @Override
            public void run() {
                Looper.prepare();
                subHandler = new Handler() {
                    @Override
                    public void handleMessage(Message msg) {
                        super.handleMessage(msg);
                        // 处理消息
                    }
                };
                Looper.loop();
            }
        }).start();
```

##### 4. handle机制

![handle机制](\time-consuming\4843132-4fb0e00953a4111d.webp)

Handler机制，主要牵涉到的类有如下四个，它们分工明确，但又相互作用

1. Message：消息
2. Hanlder：消息的发起者
3. Looper：消息的遍历者
4. MessageQueue：消息队列

###### 4.1 Looper.prepare()

```java
    public static void prepare() {
        prepare(true);
    }

    private static void prepare(boolean quitAllowed) {
        // 规定了一个线程只有一个Looper，也就是一个线程只能调用一次Looper.prepare()
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        // 如果当前线程没有Looper，那么就创建一个，存到sThreadLocal中
        sThreadLocal.set(new Looper(quitAllowed));
    }

    private Looper(boolean quitAllowed) {
        // 创建了MessageQueue，并供Looper持有
        mQueue = new MessageQueue(quitAllowed);
        // 让Looper持有当前线程对象
        mThread = Thread.currentThread();
    }
```

Looper.prepare()的作用主要有以下三点

1. 创建Looper对象
2. 创建MessageQueue对象，并让Looper对象持有
3. 让Looper对象持有当前线程

###### 4.2 new Handler()

```java
    public Handler() {
        this(null, false);
    }

    public Handler(Callback callback, boolean async) {
      // 不相关代码
       ......
        //得到当前线程的Looper，其实就是调用的sThreadLocal.get
        mLooper = Looper.myLooper();
        // 如果当前线程没有Looper就报运行时异常
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
        // 把得到的Looper的MessagQueue让Handler持有
        mQueue = mLooper.mQueue;
        // 初始化Handler的Callback，其实就是最开始图中的回调方法的2
        mCallback = callback;
        mAsynchronous = async;
    }
```

默认的Handler的创建过程主要有以下几点

1. 创建Handler对象
2. 得到当前线程的Looper对象，并判断是否为空
3. 让创建的Handler对象持有Looper、MessageQueue、Callback的引用

###### 4.3 Looper.loop()

```java
   public static void loop() {
        // 得到当前线程的Looper对象
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        // 得到当前线程的MessageQueue对象
        final MessageQueue queue = me.mQueue;
        
        // 无关代码
        ......
        
        // 死循环
        for (;;) {
            // 不断从当前线程的MessageQueue中取出Message，当MessageQueue没有元素时，方法阻塞
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }
            // Message.target是Handler，其实就是发送消息的Handler，这里就是调用它的dispatchMessage方法
            msg.target.dispatchMessage(msg);
            // 回收Message
            msg.recycleUnchecked();
        }
    }
```

```java
	//Handler的dispatchMessage方法   
	public void dispatchMessage(Message msg) {
        // 如果msg.callback不是null，则调用handleCallback
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            // 如果 mCallback不为空，则调用mCallback.handleMessage方法
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            // 调用Handler自身的handleMessage，这就是我们常常重写的那个方法
            handleMessage(msg);
        }
    }
```

可以看出，这个方法就是从MessageQueue中取出Message以后，进行分发处理。

首先，判断msg.callback是不是空，其实msg.callback是一个Runnable对象，是Handler.post方式传递进来的参数，后面会讲到。而hanldeCallback就是调用的Runnable的run方法。

然后，判断mCallback是否为空，这是一个Handler.Callback的接口类型，之前说了Handler有多个构造方法，可以提供设置Callback，如果这里不为空，则调用它的hanldeMessage方法，注意，这个方法有返回值，如果返回了true，表示已经处理 ，不再调用Handler的handleMessage方法；如果mCallback为空，或者不为空但是它的handleMessage返回了false，则会继续调用Handler的handleMessage方法，该方法就是我们经常重写的那个方法。

所以Looper.loop的作用就是：从当前线程的MessageQueue从不断取出Message，并调用其相关的回调方法。

###### 4.4 发送信息

使用Handler发送消息主要有两种，一种是sendXXXMessage方式，还有一个postXXX方式，不过两种方式最后都会调用到sendMessageDelayed方法，所以我们就以最简单的sendMessage方法来分析。

```java
    public final boolean sendMessage(Message msg)
    {
        return sendMessageDelayed(msg, 0);
    }

    public final boolean sendMessageDelayed(Message msg, long delayMillis)
    {
        if (delayMillis < 0) {
            delayMillis = 0;
        }
        return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
    }

    public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
        // 这里拿到的MessageQueue其实就是创建时的MessageQueue，默认情况是当前线程的Looper对象的MessageQueue
        // 也可以指定
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        // 调用enqueueMessage，把消息加入到MessageQueue中
        return enqueueMessage(queue, msg, uptimeMillis);
    }
```

```java
    private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        // 把当前Handler对象，也就是发起消息的handler作为Message的target属性
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        // 调用MessageQueue中的enqueueMessage方法
        return queue.enqueueMessage(msg, uptimeMillis);
    }
```

```java
  //MessageQueue的enqueueMessage方法
	boolean enqueueMessage(Message msg, long when) {
        if (msg.target == null) {
            throw new IllegalArgumentException("Message must have a target.");
        }
        // 一个Message，只能发送一次
        if (msg.isInUse()) {
            throw new IllegalStateException(msg + " This message is already in use.");
        }

        synchronized (this) {
            if (mQuitting) {
                IllegalStateException e = new IllegalStateException(
                        msg.target + " sending message to a Handler on a dead thread");
                Log.w("MessageQueue", e.getMessage(), e);
                msg.recycle();
                return false;
            }
            // 标记Message已经使用了
            msg.markInUse();
            msg.when = when;
            // 得到当前消息队列的头部
            Message p = mMessages;
            boolean needWake;
            // 我们这里when为0，表示立即处理的消息
            if (p == null || when == 0 || when < p.when) {
                // 把消息插入到消息队列的头部
                msg.next = p;
                mMessages = msg;
                needWake = mBlocked;
            } else {
                // 根据需要把消息插入到消息队列的合适位置，通常是调用xxxDelay方法，延时发送消息
                needWake = mBlocked && p.target == null && msg.isAsynchronous();
                Message prev;
                for (;;) {
                    prev = p;
                    p = p.next;
                    if (p == null || when < p.when) {
                        break;
                    }
                    if (needWake && p.isAsynchronous()) {
                        needWake = false;
                    }
                }
                // 把消息插入到合适位置
                msg.next = p; // invariant: p == prev.next
                prev.next = msg;
            }

            // 如果队列阻塞了，则唤醒
            if (needWake) {
                nativeWake(mPtr);
            }
        }
        return true;
    }
```

以上是sendMessage的全部过程，其实就是把Message加入到MessageQueue的合适位置。那我们来简单看看post系列方法：

```java
   public final boolean post(Runnable r)
   {
      return  sendMessageDelayed(getPostMessage(r), 0);
   }

   private static Message getPostMessage(Runnable r) {
       // 构造一个Message，并让其callback执行传来的Runnable
       Message m = Message.obtain();
       m.callback = r;
       return m;
   }
```

所以使用handler发送消息的本质都是：把Message加入到Handler中的MessageQueue中去。

###### 4.5 总结

我们再来总结下Handler消息机制主要的四个类的功能

1. Message:信息的携带者，持有了Handler，存在MessageQueue中，一个线程可以有多个
2. Hanlder:消息的发起者，发送Message以及消息处理的回调实现，一个线程可以有多个Handler对象
3. Looper:消息的遍历者，从MessageQueue中循环取出Message进行处理，一个线程最多只有一个
4. MessageQueue:消息队列，存放了Handler发送的消息，供Looper循环取消息，一个线程最多只有一个

##### 5.  handle的一些问题

1. Android中，有哪些是基于Handler来实现通信的？
    答：App的运行、更新UI、AsyncTask、Glide、RxJava等
2. 处理Handler消息，是在哪个线程？一定是创建Handler的线程么？
    答：创建Handler所使用的Looper所在的线程
3. 消息是如何插入到MessageQueue中的？
    答： 是根据when在MessageQueue中升序排序的，when=开机到现在的毫秒数+延时毫秒数
4. 当MessageQueue没有消息时，它的next方法是阻塞的，会导致App ANR么？
    答：不会导致App的ANR，是Linux的pipe机制保证的，阻塞时，线程挂起；需要时，唤醒线程
5. 子线程中可以使用Toast么？
    答：可以使用，但是Toast的显示是基于Handler实现的，所以需要先创建Looper，然后调用Looper.loop。
6. Looper.loop()是死循环，可以停止么？
    答：可以停止，Looper提供了quit和quitSafely方法
7. Handler内存泄露怎么解决？
    答： 静态内部类+弱引用 、Handler的removeCallbacksAndMessages等方法移除MessageQueue中的消息

##### 6.  handle内存泄露的处理

###### 6.1 静态内部类+弱引用

```java
public class HandlerActivity extends AppCompatActivity {
    private static final String TAG = "HandlerActivity";
    private Handler mHandler;
    private Button btn;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_handler);
        
        // 创建Handler对象，把Activity对象传入
        mHandler = new MyHandler(HandlerActivity.this);

        btn = (Button) findViewById(R.id.btn);
        btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // 发送延时100s的消息
                mHandler.sendEmptyMessageDelayed(100, 100 * 1000);
            }
        });
    }

    // 静态内部类
    static class MyHandler extends Handler {
        private WeakReference<Activity> activityWeakReference;
        public  MyHandler(Activity activity) {
            activityWeakReference = new WeakReference<Activity>(activity);
        }
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            // 处理消息
            if (activityWeakReference != null) {
                Activity activity = activityWeakReference.get();
                // 拿到activity对象以后，调用activity的方法
                if (activity != null) {

                }
            }
        }
    }
}
```

首先，我们自定义了一个静态内部类MyHandler，然后创建MyHandler对象时传入当前Activity的对象，供Hander以弱应用的方式持有，这个时候Activity就被强引用和弱引用两种方式引用了，我们继续发起一个延时100s的消息，然后退出当前Activity，这个时候Activity的强引用就不存在了，只存在弱引用，gc运行时会回收掉只有弱引用的Activity，这样就不会造成内存泄漏了。

但这个延时消息还是存在于MessageQueue中，得到这个Message被取出时，还是会进行分发处理，只是这时候Activity被回收掉了，activity为null，不能再继续调用Activity的方法了。所以，其实这是Activity可以被回收了，而Handler、Message都不能被回收。

至于为什么使用弱引用而没有使用软引用，其实很简单，对比下两者回收前提条件就清楚了

1. 弱引用(WeakReference): gc运行时，无论内存是否充足，只有弱引用的对象就会被回收
2. 软引用(SoftReference): gc运行时，只有内存不足时，只有软引用的对象就会被回收

很明显，当我们Activity退出时，我们希望不管内存是否足够，都应该回收Activity对象，所以使用弱引用合适。

###### 6.2 移除MessageQueue中的消息

我们知道，内存泄漏的源头是MessageQueue持有的Message持有了Handler持有了Activity，那我们在合适的地方把Message从MessageQueue中移除，不就可以解决内存泄漏了么？

Handler为我们提供了removeCallbacksAndMessages等方法用于移除消息，比如，在Activity的onDestroy中调用Handler的removeCallbacksAndMessages，代码如下：



```java
    @Override
    protected void onDestroy() {
        super.onDestroy();
        // 移除MessageQueue中target为该mHandler的Message
        mHandler.removeCallbacksAndMessages(null);
    }
```

其实就是在Activity的onDestroy方法中调用mHandler.removeCallbacksAndMessages(null)，这样就移除了MessageQueue中target为该mHandler的Message，因为MessageQueue没有引用该Handler发送的Message了，所以当Activity退出时，Message、Handler、Activity都是可回收的了，这样就能解决内存泄漏的问题了。

#### ③ Thread

**实现原理：**

创建一个Thread对象，然后在其run方法中调用runOnUiThread方法，在run中执行耗时操作，在runOnUiThread方法执行耗时操作完成后需要更新的UI，不要忘记调用Thread的start方法。

```java
	new Thread(new Runnable() {
//	开启一个线程处理逻辑，然后在线程中在开启一个UI线程，当子线程中的逻辑完成之后，
//	就会执行UI线程中的操作，将结果反馈到UI界面。
		@Override
		public void run() {
			// 模拟耗时的操作，在子线程中进行。
			try {
				Thread.sleep(5000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			// 更新主线程ＵＩ，跑在主线程。
			TestActivity.this.runOnUiThread(new Runnable() {
				@Override
				public void run() {
					// 更新UI
				}
			});
		}
	}).start();
```

​	