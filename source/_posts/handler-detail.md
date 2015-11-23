title: Android Handler详解
date: 2015-06-10 23:00:49
categories: Android
tags: [Android,handler]
---
### Android为什么要设计只能通过Handler机制更新UI？

最根本的问题就是解决多线程并发问题，假设如果在一个Activity当中，有多个线程去更新UI,并且没有加锁机制，那将造成界面更新错乱。倘若如果对更新UI的操作都进行加锁处理又将导致性能的下降。

处于对以上问题的考虑，android 给我们提供了一套UI更新的机制，我们只需遵循这样的机制就可以了，不用担心多线程的问题，更新UI的操作都是在主线程的消息队列中去轮询处理的。

### Handler 原理是什么？
Android上一个应用的入口是ActivityThread，和普通的java类一样，入口是一个main方法。在其中会给我们创建一个Looper，创建Looper的过程中会创建一个MessageQueue对象。
<!-- more -->
> ActivityThread main()方法
```java
public static void main(String[] args) {  
        SamplingProfilerIntegration.start();  
  
        CloseGuard.setEnabled(false);  
  
        Environment.initForCurrentUser();  
  
        // Set the reporter for event logging in libcore  
        EventLogger.setReporter(new EventLoggingReporter());  
  
        Process.setArgV0("<pre-initialized>");  
  
        Looper.prepareMainLooper();  
  
        // 创建ActivityThread实例  
        ActivityThread thread = new ActivityThread();  
        thread.attach(false);  
  
        if (sMainThreadHandler == null) {  
            sMainThreadHandler = thread.getHandler();  
        }  
  
        AsyncTask.init();  
  
        if (false) {  
            Looper.myLooper().setMessageLogging(new LogPrinter(Log.DEBUG, "ActivityThread"));  
        }  
  
        Looper.loop();  
  
        throw new RuntimeException("Main thread loop unexpectedly exited");  
    }  
}  
```

> Looper
```java
    public static void prepareMainLooper() {
        prepare(false);
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
    }
    
    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        // 创建Looper对象
        sThreadLocal.set(new Looper(quitAllowed));
    }
    
    private Looper(boolean quitAllowed) {
        // 创建MessageQueue 对象
        mQueue = new MessageQueue(quitAllowed);
        mRun = true;
        mThread = Thread.currentThread();
    }
    
    public static Looper myLooper() {
        return sThreadLocal.get();
    }
    
    public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }

            // msg 回传给自己
            msg.target.dispatchMessage(msg);

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            msg.recycle();
        }
    }

```
> Handler
```java
   public Handler(){
        ······
        
        mLooper = Looper.myLooper();
        
        ······
        mQueue = mLooper.mQueue;
        mCallBack = null;
    }

    public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, uptimeMillis);
    }
    
    private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }
    
    public void dispatchMessage(Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }
```

1. Looper(消息载体)：内部包含一个消息队列也就是MessageQueue，所有的Handler发送的消息都走向这个消息队列，Looper.loop方法，就是一个死循环，不断的从MessageQueue取消息，有消息就处理消息，没有消息就阻塞。
2. MessageQueue(消息容器)：一个消息队列，可以添加消息。
3. Handler内部和Looper进行关联，也就是说在Handler中可以找到Looper，找到了Looper也就找到了MessageQueue，在Handler中发送消息其实就是向MessageQueue队列中发送消息，Looper负责接收Handler发送的消息，并直接把消息回传给Handler自己。

### 指定线程处理消息 -- handleMessage
在Handler构造方法中可指定某个线程Looper来作为本身消息的轮询处理。其中HandlerThread继承自Thread，维护了一个所在线程的一个Looper，handlerThread.getLooper()即可获得子线程的一个Looper。
```java
    new Thread(){
    	@Override
    	public void run() {
    		
    		Handler handler = new Handler(Looper.getMainLooper()){
    			public void handleMessage(android.os.Message msg) {
    				System.out.println("子线程创建的handler在: " + Thread.currentThread() + " 接收消息");
    			};
    		};
    		
    		handler.sendEmptyMessage(0);
    		
    	}
    }.start();
    
    HandlerThread handlerThread = new HandlerThread("handler thread");
    handlerThread.start();
    
    Handler handler = new Handler(handlerThread.getLooper()){
    	@Override
    	public void handleMessage(Message msg) {
    		System.out.println("主线程创建的handler在: " + Thread.currentThread() + " 接收消息");
    	}
    };
    
    handler.sendEmptyMessage(0);
```

### Android中更新UI的几种方法
* runOnUIThread
```java
    public final void runOnUiThread(Runnable action) {
        if (Thread.currentThread() != mUiThread) {
            mHandler.post(action);
        } else {
            action.run();
        }
    }
```
* handler post
```java
    public final boolean post(Runnable r){
       return sendMessageDelayed(getPostMessage(r), 0);
    }
    
    private static Message getPostMessage(Runnable r) {
        Message m = Message.obtain();
        m.callback = r;
        return m;
    }
    
    public final boolean sendMessageDelayed(Message msg, long delayMillis){
        if (delayMillis < 0) {
            delayMillis = 0;
        }
        return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
    }
```
* handler sendMessage
```java
    public final boolean sendMessage(Message msg){
        return sendMessageDelayed(msg, 0);
    }
```
* view post
```java
    /**
     * <p>Causes the Runnable to be added to the message queue.
     * The runnable will be run on the user interface thread.</p>
     *
     * @param action The Runnable that will be executed.
     *
     * @return Returns true if the Runnable was successfully placed in to the
     *         message queue.  Returns false on failure, usually because the
     *         looper processing the message queue is exiting.
     *
     */
    public boolean post(Runnable action) {
        final AttachInfo attachInfo = mAttachInfo;
        if (attachInfo != null) {
            return attachInfo.mHandler.post(action);
        }
        // Assume that post will succeed later
        ViewRootImpl.getRunQueue().post(action);
        return true;
    }
```