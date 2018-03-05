#Android BroadcastReveiver   
##认识  
BroadcastReceiver，广播接收者，它是一个系统全局的监听器，用于监听系统全局的Broadcast消息，所以它可以很方便的进行系统组件之间的通信。  
BroadcastReceiver属于系统级的监听器，它拥有自己的进程，只要存在与之匹配的Broadcast被以Intent的形式发送出来，BroadcastReceiver就会被激活。  
虽然同属Android的四大组件，BroadcastReceiver也有自己独立的声明周期，但是和Activity、Service又不同。当在系统注册一个BroadcastReceiver之后，每次系统以一个Intent的形式发布Broadcast的时候，系统都会创建与之对应的BroadcastReceiver广播接收者实例，并自动触发它的onReceive()方法，当onReceive()方法被执行完成之后，BroadcastReceiver的实例就会被销毁。虽然它独自享用一个单独的进程，但也不是没有限制的，如果BroadcastReceiver.onReceive()方法不能在10秒内执行完成，Android系统就会认为该BroadcastReceiver对象无响应，然后弹出ANR（Application No Response）对话框，所以不要在BroadcastReceiver.onReceive()方法内执行一些耗时的操作。  
如果需要根据广播内容完成一些耗时的操作，一般考虑通过Intent启动一个Service来完成该操作，而不应该在BroadcastReceiver中开启一个新线程完成耗时的操作，因为BroadcastReceiver本身的生命周期很短，可能出现的情况是子线程还没有结束，BroadcastReceiver就已经退出的情况，而如果BroadcastReceiver所在的进程结束了，该线程就会被标记为一个空线程，根据Android的内存管理策略，在系统内存紧张的时候，会按照优先级，结束优先级低的线程，而空线程无异是优先级最低的，这样就可能导致BroadcastReceiver启动的子线程不能执行完成。  
使用时首先，继承 BroadcastReceiver 类创建一个用于接收标准广播的Receiver，重写onReceive方法
## Broadcast的种类  
广播有两种，决定发送那种广播的地方是发送广播的时候调用的是sendBroadcast(Intent)还是sendOrderedBroadcast(Intent,String)  

* Normal Broadcast（普通广播）：一种完全异步执行的广播，在广播发出后所有的广播接收器会在同一时间接收到这条广播，之间没有先后顺序，效率比较高，且无法被截断。  
* Ordered Broadcast（有序广播）：一种同步执行的广播，在广播发出后同一时刻只有一个广播接收器能够接收到， 优先级高的(数值越大优先级越高)广播接收器会优先接收，当优先级高的广播接收器的 onReceiver() 方法运行结束后，广播才会继续传递，且前面的广播接收器可以选择截断广播，这样后面的广播接收器就无法接收到这条广播了
  
>1，该广播的级别有级别之分，级别数值是在-1000到1000之间,值越大,优先级越高；  
2，同级别接收是先后是随机的，再到级别低的收到广播；  
3，同级别接收是先后是随机的，如果先接收到的把广播截断了，同级别的例外的接收者是无法收到该广播的。（abortBroadcast()）  
4，能截断广播的继续传播，高级别的广播收到该广播后，可以决定把该钟广播是否截断掉(**BroadcastReceiver 也能调用 abortBroadcast() 方法截断广播，这样低优先级的广播接收器就无法接收到广播了**)。  
5，实验现象，在这个方法发来的广播中，代码注册方式中，收到广播先后次序为：注明优先级的、代码注册的、没有优先级的；如果都没有优先级，代码注册收到为最先。

---
>注意：这里是广播的种类，不干扰我们自定义接收器的流程。
###一 静态注册  
静态注册即在**清单文件中为 BroadcastReceiver 进行注册**，使用**< receiver >**标签声明，并在标签内用 **< intent-filter > **标签设置过滤器。这种形式的 BroadcastReceiver 的生命周期伴随着整个应用，**如果这种方式处理的是系统广播，那么不管应用是否在运行，该广播接收器都能接收到该广播**。广播的优先级别通过intent-filter中的priority属性来指定：  

	<application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

		
        <receiver android:name=".OrderReceiver_1">
            <intent-filter android:priority="100">
                <action android:name="com.example.order.receiver" />
            </intent-filter>
        </receiver>
		
        <receiver android:name=".OrderReceiver_2">
            <intent-filter android:priority="99">
                <action android:name="com.example.order.receiver" />
            </intent-filter>
        </receiver>
		
        <receiver android:name=".OrderReceiver_3">
            <intent-filter android:priority="98">
                <action android:name="com.example.order.receiver" />
            </intent-filter>
        </receiver>
		
    </application>

### 二 动态注册  
动态注册 BroadcastReceiver 是在代码中定义并设置好一个 IntentFilter 对象，然后在需要注册的地方调用 **Context.registerReceiver() **方法，调用 **Context.unregisterReceiver() **方法取消注册，此时就不需要在清单文件中注册 Receiver 了

>对于动态广播，有注册就必然得有注销，否则会导致内存泄露;重复注册、重复注销也不允许  

## 本地广播  
之前发送和接收到的广播全都是属于系统全局广播，即发出的广播可以被其他应用接收到，而且也可以接收到其他应用发送出的广播，这样可能会有不安全因素  
因此，在某些情况下可以采用本地广播机制，使用这个机制发出的广播只能在应用内部进行传递，而且广播接收器也只能接收本应用内自身发出的广播  
本地广播是使用 **LocalBroadcastManager**来对广播进行管理  

* LocalBroadcastManager.getInstance(this).registerReceiver(BroadcastReceiver, IntentFilter)：注册Receiver  
* LocalBroadcastManager.getInstance(this).unregisterReceiver(BroadcastReceiver)：注销Receiver  
* LocalBroadcastManager.getInstance(this).sendBroadcast(Intent)：发送异步广播  
* LocalBroadcastManager.getInstance(this).sendBroadcastSync(Intent)：发送同步广播

## 使用私有权限  
使用动态注册广播接收器存在一个问题，即系统内的任何应用均可监听并触发我们的 Receiver 。通常情况下我们是不希望如此的  
解决办法之一是在清单文件中为 **< receiver > 标签添加一个 android:exported="false"** 属性，标明该 Receiver 仅限应用内部使用。这样，系统中的其他应用就无法接触到该 Receiver 了  
此外，也可以选择创建自己的使用权限，即在清单文件中添加一个 **< permission > 标签来声明自定义权限**  

	<permission
        android:name="com.example.permission.receiver"
        android:protectionLevel="signature" />  

自定义权限时必须同时指定 protectionLevel 属性值，系统根据该属性值确定自定义权限的使用方式 ，属性值有：  

* normal：默认值。较低风险的权限，对其他应用，系统和用户来说风险最小。系统在安装应用时会自动批准授予应用该类型的权限，不要求用户明确批准（虽然用户在安装之前总是可以选择查看这些权限）  
* dangerous：较高风险的权限，请求该类型权限的应用程序会访问用户私有数据或对设备进行控制，从而可能对用户造成负面影响。因为这种类型的许可引入了潜在风险，所以系统可能不会自动将其授予请求的应用。例如，系统可以向用户显示由应用请求的任何危险许可，并且在继续之前需要确认，或者可以采取一些其他方法来避免用户自动允许  
* signature：只有在请求该权限的应用与声明权限的应用使用相同的证书签名时，系统才会授予权限。如果证书匹配，系统会自动授予权限而不通知用户或要求用户的明确批准  
* signatureOrSystem：系统仅授予Android系统映像中与声明权限的应用使用相同的证书签名的应用。请避免使用此选项，“signature”级别足以满足大多数需求，“signatureOrSystem”权限用于某些特殊情况  



作者：叶应是叶
链接：https://juejin.im/post/5a9266185188257a5a4ccfc5
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。   


作者：叶应是叶
链接：https://juejin.im/post/5a9266185188257a5a4ccfc5
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
作者：叶应是叶
链接：https://juejin.im/post/5a9266185188257a5a4ccfc5
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
作者：叶应是叶
链接：https://juejin.im/post/5a9266185188257a5a4ccfc5
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。