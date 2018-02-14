# Handler详解  
提到Handler就不得不提Looper和MessageQueue了，他们之间的关系如下：  

* Handler负责消息的发送和处理：Handler发送消息给MessageQueue和接收Looper返回的消息并且处理消息  
* Looper负责管理MessageQueue：Looper会不断的从MessageQueue中取出消息，交给Handler处理  
* MessageQueue是消息队列（实际上是用链表实现的），负责存放Handler发送过来的消息  
* 一个Looper对应一个线程  

接下来就详细的说一说他们之间的交互是怎么进行的。  
首先从Handler的构造器开始：  

	public Handler() {
        this(null, false);
    }

    public Handler(Callback callback, boolean async) {
        //获取Looper对象
        mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
        //获取Looper对象的mQueue属性，mQueue 就是MessageQueue对象。
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }  
>在源码中，其构造器的参数还可以带一个Looper，不管其他形式的构造器如何，本质都是调用的Looper(Callback,boolean)这个构造器；带Looper的构造器将**mLooper=Looper.mylooper()换成了mLooper=looper**

可以看到，**Handler进行交互的MessageQueue其实是Looper中的MessageQueue。**而首先获取Looper对象的方法是Looper的mylooper（）静态方法，如果这时候抛出异常说明没有执行Looper.prepare()方法。接下来看看Looper.mylooper()和Looper.prepare()方法：  

	static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();

    //创建当前线程的Looper对象
    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
		//此处调用了私有的构造器
        sThreadLocal.set(new Looper(quitAllowed));
    }

    //获取当前线程的Looper对象
    public static @Nullable Looper myLooper() {
        return sThreadLocal.get();
    }  
这里有一个很关键的类：**ThreadLocal**，它一个线程内部的数据存储类，通过它存储的数据只有在它自己的线程才能获取到，其他线程是获取不到的。所以sThreadLocal.get()获取的就是当前线程的Looper对象。在Looper.prepare()方法中我们看到了如果当前线程已经有Looper对象，就会抛出异常，说一个线程只能创建一个Looper对象，所以Looper对象与自己所在的线程是相对应的。  
再看看Looper的构造方法：  

	private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }  
可见，在这个私有构造器中Looper初始化了一个**MessageQueue**并且**将自己绑定到了当前的线程中**  
Handler发送消息的方法有很多，但无论你是send一个Message还是post一个Runnable；无论你是延时发送还是不延时发送，最终都会调用Handler的enqueueMessage()方法。  

	private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        //把this赋值给msg的target属性，this就是Handler对象。
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        //把消息存放到MessageQueue 
        return queue.enqueueMessage(msg, uptimeMillis);
    }  
这里是直接把消息放到了MessageQueue就完事了。开头说了**Looper是不断从MessageQueue中取出Message交给Handler来处理的，这个过程就是通过loop()方法来实现的**  

	public static void loop() {

        final MessageQueue queue = me.mQueue;
        //一个死循环
        for (;;) {
            //从MessageQueue中取出一条消息
            Message msg = queue.next(); 
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }
            //把消息交给Handler处理。
            msg.target.dispatchMessage(msg);
        }
    }  
从上面的代码中我们看到loop()会开启一个死循环，不断地从MessageQueue中取出消息并交给Handler处理。在前面的enqueueMessage()方法中我们知道了msg.target就是发送消息的Handler对象。
这里有同学可能会有疑问：上面的代码中明明如果(msg == null)，就退出方法，为什么我还说loop()里面是个死循环呢？这是因为MessageQueue的next()方法取出消息的时候，如果没有消息，next()方法会阻塞线程，直到MessageQueue有消息进来，然后取出消息并返回。所以queue.next()一般不会返回null，除非调用Looper的quit()或者quitSafely()方法结束消息轮询，queue.next()才会返回null，才会结束循环。  

	public void quit() {
        mQueue.quit(false);
    }
    public void quitSafely() {
        mQueue.quit(true);
    }  
最后我们来看 一下消息的处理：Handler的dispatchMessage(msg)方法  

	public void dispatchMessage(Message msg) {
        //如果Message有自己的callback，就由Message的callback处理
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
             //如果Handler有自己的mCallback，就由Handler的mCallback处理
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            //默认的处理消息的方法
            handleMessage(msg);
        }
    }  
处理消息的方法有三个：
1、优先级最高的是Message自己的callback，这是一个Runnable对象，我们用Handler post一个Runnable的时候，其实就是把这个Runnable赋值个一个Message对象的callback，然后把Message对象发送给MessageQueue。
2、优先级第二的是Handler自己的mCallback，在我们创建一个Handler对象的使用可以传一个Handler.Callback对象给它并实现Callback里的handleMessage(msg)方法，作为我们的消息处理方法。
3、优先级最低的是Handler自己的handleMessage(msg)方法，这也是我们最常用的消息处理方法。  

回顾一下我们使用Handler的几个步骤：   

1. Looper.prepare()方法：该方法的作用是  
	* 内部实例化一个looper对象  
	* 初始化MessageQueue  
	* 绑定到当前线程  
2. 调用Handler的构造器实例化一个对象并重写handleMessage方法  
	* 绑定了该线程的Looper  
	* 从Looper处得到了MessageQueue  
3. 调用Looper.loop()方法  
	* 开始发送消息给Handler处理

# HandlerThread详解  
HandlerThread是安卓内部封装好的一个**轻量级的线程**，和Handler配合使用实现耗时操作或者异步操作。  
### 源码分析  

	public class HandlerThread extends Thread {//继承自Thread类
    int mPriority;
    int mTid = -1;
    Looper mLooper;//内部维护一个Looper

    public HandlerThread(String name) {
        super(name);
        mPriority = Process.THREAD_PRIORITY_DEFAULT;//线程优先级用的默认优先级
    }
    
    public HandlerThread(String name, int priority) {
        super(name);
        mPriority = priority;
    }

	protected void onLooperPrepared() { //用户可以重写该方法，在Looper.loop()之前做一些准备工作
    }

    @Override
    public void run() {//线程体执行的内容
        mTid = Process.myTid();
        Looper.prepare();
        synchronized (this) {
            mLooper = Looper.myLooper();//获得一个Looper
            notifyAll();
        }
        Process.setThreadPriority(mPriority);
        onLooperPrepared();//hook onLooperPrepared
        Looper.loop();//启动消息循环
        mTid = -1;
    }

	public Looper getLooper() {
        if (!isAlive()) {
            return null;
        }
        synchronized (this) {
            while (isAlive() && mLooper == null) {
                try {
                    wait();
                } catch (InterruptedException e) {
                }
            }
        }
        return mLooper;
    }

   
    public boolean quit() {
        Looper looper = getLooper();
        if (looper != null) {
            looper.quit();
            return true;
        }
        return false;
    }

	public boolean quitSafely() {
        Looper looper = getLooper();
        if (looper != null) {
            looper.quitSafely();
            return true;
        }
        return false;
    }

    public int getThreadId() {
        return mTid;
    }
	}

通过源码我们可以看出HandlerThread的内部管理了一个Looper。  

>**run方法分析：**  
>首先调用了Looper.prepare()方法  
>然后将属性mLooper指向Looper.myLooper()的返回值  
>其次**执行onLooperPreoared()方法，开发者根据需求可以重写，做一些想要的操作**  
>最后执行了Looper.loop()方法发送消息  

**结合Handler与HandlerThread的使用方法是，初始化Handler的时候需要重写其handleMessage(Message msg)方法，构造器的参数为HandlerThread对象的getLooper()方法的返回值，这样与Handler绑定的线程就是HandlerThread了，所有事情做完后执行HandlerThread对象的quit方法关闭消息循环.**  

### IntentService详解  
IntentService也是Android官方封装的一个Service**抽象类**，内部是基于HandlerThread实现的，  

#### 源代码  

	public abstract class IntentService extends Service {
    private volatile Looper mServiceLooper;
    private volatile ServiceHandler mServiceHandler;//内部使用了一个Handler
    private String mName;
    private boolean mRedelivery;

    private final class ServiceHandler extends Handler {
        public ServiceHandler(Looper looper) {
            super(looper);
        }

        @Override
        public void handleMessage(Message msg) {
            onHandleIntent((Intent)msg.obj);//收到消息后，回调onHandleIntent()方法 注意该方法的调用是在非主线程中调用的
            stopSelf(msg.arg1);//待线程体执行结束后，结束该服务
        }
    }

	public IntentService(String name) {
        super();
        mName = name;
    }

  
    public void setIntentRedelivery(boolean enabled) {
        mRedelivery = enabled;
    }

    @Override
    public void onCreate() {
      
        super.onCreate();
        HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");//在Service
 	OnCreate中，使用HandlerThread 启动一个带Looper的线程
        thread.start();

        mServiceLooper = thread.getLooper();//获取到Looper
        mServiceHandler = new ServiceHandler(mServiceLooper);//生成一个Handler ，前边讲过，Handler的生成需要Looper。
    }

	@Override
    public void onStart(@Nullable Intent intent, int startId) {
        Message msg = mServiceHandler.obtainMessage();
        msg.arg1 = startId;
        msg.obj = intent;
        mServiceHandler.sendMessage(msg);//在onStart()方法中发送一个消息
    }

  
    @Override
    public int onStartCommand(@Nullable Intent intent, int flags, int startId) {
        onStart(intent, startId);
        return mRedelivery ? START_REDELIVER_INTENT : START_NOT_STICKY;
    }

    @Override
    public void onDestroy() {
        mServiceLooper.quit();
    }

 
    @Override
    @Nullable
    public IBinder onBind(Intent intent) {
        return null;
    }

  
    @WorkerThread
    protected abstract void onHandleIntent(@Nullable Intent intent);//重写方法 讲耗时的内容在该方法体实现
	}

可以看到，IntentService内部就是通过Handler和HandlerThread共同协作实现的功能。  
想要使用该Service，需要创建一个实现类，去实现功能。