## Service解析

> *熟悉了activity，不得不提到android的第二大组件—Service，如果说activity是面子工程，service可以算作是里子工程了。这篇文章主要介绍一下service的使用*

## 一. service简介

Service(服务)是一个一种可以在后台执行长时间运行操作而没有用户界面的应用组件。服务可由其他应用组件启动（如Activity），服务一旦被启动将在后台一直运行，即使启动服务的组件（Activity）已销毁也不受影响。 此外，组件可以绑定到服务，以与之进行交互，甚至是执行进程间通信 (IPC)。 例如，服务可以处理网络事务、播放音乐，执行文件 I/O 或与内容提供程序交互，而所有这一切均可在后台进行。

- 定义：服务，属于Android中的计算型组件
- 作用：提供需要在后台长期运行的服务（如复杂计算、下载等等）
- 特点：长生命周期的、没有用户界面、在后台运行

## 二. 生命周期

首先看一下官方给出的生命周期图

![](<image/servicelife.png>)

- **onCreate()**

  首次创建服务时，系统将调用此方法来执行**一次性设置**程序（在调用 onStartCommand() 或onBind() 之前）。如果服务已在运行，则不会调用此方法，该方法只调用一次

- **onStartCommand()**

  当另一个组件（如 Activity）通过调用 startService() 请求启动服务时，系统将调用此方法。一旦执行此方法，服务即会启动并可在**后台无限期运行**。 如果自己实现此方法，则需要在服务工作完成后，通过调用 stopSelf() 或 stopService() 来停止服务。（在绑定状态下，无需实现此方法。）

- **onBind()**

  当另一个组件想通过调用 bindService() 与服务绑定（例如执行 RPC）时，系统将调用此方法。在此方法的实现中，必须返回 一个IBinder 接口的实现类，供客户端用来与服务进行通信。无论是启动状态还是绑定状态，此方法必须重写，但在启动状态的情况下直接返回 null。

- **onDestroy()**

  当服务不再使用且将被销毁时，系统将调用此方法。服务应该实现此方法来清理所有资源，如线程、注册的侦听器、接收器等，这是服务接收的最后一个调用。

- **onUnbind()**

  调用者与服务解除绑定时调用

​	一旦在项目的任何位置调用了 Context的 **startService**()方法，相应的服务就会启动起来，并回调 **onStartCommand**()方法。如果这个服务之前还没有创建过，**onCreate**()方法会先于**onStartCommand**()方法执行。服务启动了之后会一直保持运行状态，直到 **stopService**()或**stopSelf**()方法被调用。注意虽然每调用一次**startService**()方法，**onStartCommand**()就会执行一次，但实际上每个服务都只会存在一个实例。所以不管你调用了多少次 startService()方法， 只需调用一次 stopService()或 stopSelf()方法，服务就会停止下来了。

​	另外，还可以调用 Context的**bindService()**来获取一个服务的持久连接，这时就会回调服务中的 onBind()方法。类似地，如果这个服务之前还没有创建过，onCreate()方法会先于 onBind()方法执行。之后，调用方可以获取到 onBind()方法里返回的 IBinde对象的实例，这样就能自由地和服务进行通信了。只要调用方和服务之间的连接没有断开，服务就会一直保持运行状态。 当调用了 startService()方法后，又去调用 stopService()方法，这时服务中的 onDestroy() 方法就会执行，表示服务已经销毁了。类似地，当调用了 bindService()方法后，又去调用 unbindService()方法，onDestroy()方法也会执行，这两种情况都很好理解。

 	但是需要注意，我们是完全有可能对一个服务既调用了 startService()方法，又调用了 bindService()方法的， 这种情况下该如何才能让服务销毁掉呢？根据 Android系统的机制，一个服务只要被启动或者被绑定了之后，就会一直处于运行状态，必须要让以上两种条件同时不满足，服务才能被销毁。所以，这种情况下要同时调用 stopService()和 unbindService()方法，onDestroy()方法才会执行。

## 三. service的分类

### 1. Service的类型

service在不同的场景下，对应的服务也不相同，将服务进行分类

![](<image/service_sort.png>)

### 2. 特点

![](<image/service_ feature.png>)

## 四.具体使用

### 1. 本地Service

- **步骤1：新建子类继承Service类**

> 需重写父类的onCreate()、onStartCommand()、onDestroy()和onBind()

*MyService.java*

```
public class MyService extends Service {
//启动Service之后，就可以在onCreate()或onStartCommand()方法里去执行一些具体的逻辑
//由于这里作Demo用，所以只打印一些语句
    @Override
    public void onCreate() {
        super.onCreate();
        System.out.println("执行了onCreat()");
    }
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        System.out.println("执行了onStartCommand()");
        return super.onStartCommand(intent, flags, startId);
    }
    @Override
    public void onDestroy() {
        super.onDestroy();
        System.out.println("执行了onDestory()");
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }
}
```

- **步骤2：构建Intent对象，并调用startService()启动Service、stopService停止服务**

***MainActivity.java***

```
public class MainActivity extends AppCompatActivity implements View.OnClickListener {

    private Button startService;
    private Button stopService;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        startService = (Button) findViewById(R.id.startService);
        stopService = (Button) findViewById(R.id.stopService);

        startService.setOnClickListener(this);
        startService.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            //点击启动Service Button
            case R.id.startService:
                //构建启动服务的Intent对象
                Intent startIntent = new Intent(this, MyService.class);
                //调用startService()方法-传入Intent对象,以此启动服务
                startService(startIntent);
            //点击停止Service Button
            case R.id.stopService:
                //构建停止服务的Intent对象
                Intent stopIntent = new Intent(this, MyService.class);
                //调用stopService()方法-传入Intent对象,以此停止服务
                stopService(stopIntent);
        }
    }
}
```

- **步骤3：在AndroidManifest.xml里注册Service**

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="scut.carson_ho.demo_service">
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
        //注册Service服务
        <service android:name=".MyService">
        </service>
    </application>
</manifest>
```

### 2.可通信的服务Service

上面介绍的Service是最基础的，但只能单机使用，即无法与Activity通信

接下来将在上面的基础用法上，增设“与Activity通信”的功能，即使用绑定Service服务（Binder类、bindService()、onBind(）、unbindService()、onUnbind()）

- **步骤1：在新建子类继承Service类，并新建一个子类继承自Binder类、写入与Activity关联需要的方法、创建实例**

```
public class MyService extends Service {

    private MyBinder mBinder = new MyBinder();

    @Override
    public void onCreate() {
        super.onCreate();
        System.out.println("执行了onCreat()");
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        System.out.println("执行了onStartCommand()");
        return super.onStartCommand(intent, flags, startId);

    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        System.out.println("执行了onDestory()");
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        System.out.println("执行了onBind()");
        //返回实例
        return mBinder;
    }

    @Override
    public boolean onUnbind(Intent intent) {
        System.out.println("执行了onUnbind()");
        return super.onUnbind(intent);
    }

    //新建一个子类继承自Binder类
    class MyBinder extends Binder {

        public void service_connect_Activity() {
            System.out.println("Service关联了Activity,并在Activity执行了Service的方法");

        }
    }
}
```

**步骤2：在Activity通过调用MyBinder类中的public方法来实现Activity与Service的联系**

> 即实现了Activity指挥Service干什么Service就去干什么的功能

*MainActivity.java*

```
public class MainActivity extends AppCompatActivity implements View.OnClickListener {

    private Button startService;
    private Button stopService;
    private Button bindService;
    private Button unbindService;

    private MyService.MyBinder myBinder;

    //创建ServiceConnection的匿名类
    private ServiceConnection connection = new ServiceConnection() {

        //重写onServiceConnected()方法和onServiceDisconnected()方法
        //在Activity与Service建立关联和解除关联的时候调用
        @Override
        public void onServiceDisconnected(ComponentName name) {
        }
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            //实例化Service的内部类myBinder
            //通过向下转型得到了MyBinder的实例
            myBinder = (MyService.MyBinder) service;
            //在Activity调用Service类的方法
            myBinder.service_connect_Activity();
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        startService = (Button) findViewById(R.id.startService);
        stopService = (Button) findViewById(R.id.stopService);

        startService.setOnClickListener(this);
        stopService.setOnClickListener(this);

        bindService = (Button) findViewById(R.id.bindService);
        unbindService = (Button) findViewById(R.id.unbindService);

        bindService.setOnClickListener(this);
        unbindService.setOnClickListener(this);

    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {

            //点击启动Service
            case R.id.startService:
                //构建启动服务的Intent对象
                Intent startIntent = new Intent(this, MyService.class);
                //调用startService()方法-传入Intent对象,以此启动服务
                startService(startIntent);
                break;

            //点击停止Service
            case R.id.stopService:
                //构建停止服务的Intent对象
                Intent stopIntent = new Intent(this, MyService.class);
                //调用stopService()方法-传入Intent对象,以此停止服务
                stopService(stopIntent);
                break;

            //点击绑定Service
            case R.id.bindService:
                //构建绑定服务的Intent对象
                Intent bindIntent = new Intent(this, MyService.class);
                //调用bindService()方法,以此停止服务

                bindService(bindIntent,connection,BIND_AUTO_CREATE);
                //参数说明
                //第一个参数:Intent对象
                //第二个参数:上面创建的Serviceconnection实例
                //第三个参数:标志位
                //这里传入BIND_AUTO_CREATE表示在Activity和Service建立关联后自动创建Service
                //这会使得MyService中的onCreate()方法得到执行，但onStartCommand()方法不会执行
                break;

            //点击解绑Service
            case R.id.unbindService:
                //调用unbindService()解绑服务
                //参数是上面创建的Serviceconnection实例
                unbindService(connection);
                break;

                default:
                    break;

        }
    }
}
```

**注意：onStartCommand()方法不会执行**

### 3. 前台Service

前台Service和后台Service（普通）最大的区别就在于：

- 前台Service在下拉通知栏有显示通知，但后台Service没有；
- 前台Service优先级较高，不会由于系统内存不足而被回收；后台Service优先级较低，当系统出现内存不足情况时，很有可能会被回收

用法很简单，只需要在原有的Service类对onCreate()方法进行稍微修改即可，

```
@Override
    public void onCreate() {
        super.onCreate();
        System.out.println("执行了onCreat()");

        //添加下列代码将后台Service变成前台Service
        //构建"点击通知后打开MainActivity"的Intent对象
        Intent notificationIntent = new Intent(this,MainActivity.class);
        PendingIntent pendingIntent = PendingIntent.getActivity(this,0,notificationIntent,0);

        //新建Builer对象
        Notification.Builder builer = new Notification.Builder(this);
        builer.setContentTitle("前台服务通知的标题");//设置通知的标题
        builer.setContentText("前台服务通知的内容");//设置通知的内容
        builer.setSmallIcon(R.mipmap.ic_launcher);//设置通知的图标
        builer.setContentIntent(pendingIntent);//设置点击通知后的操作

        Notification notification = builer.getNotification();//将Builder对象转变成普通的notification
        startForeground(1, notification);//让Service变成前台Service,并在系统的状态栏显示出来

    }
```

### 4. 远程service

主要和ALDL和IPC相关，这里暂时不做介绍

## 五. 其他说明

### 1. **Service和线程的关系：**

Service是运行在主线程里的，也就是说如果你在Service里编写了非常耗时的代码，程序必定会出现ANR的。

你可能会惊呼，这不是坑爹么！？那我要Service又有何用呢？其实大家不要把后台和子线程联系在一起就行了，这是两个完全不同的概念。**Android的后台就是指，它的运行是完全不依赖UI的**。即使Activity被销毁，或者程序被关闭，只要进程还在，Service就可以继续运行。比如说一些应用程序，始终需要与服务器之间始终保持着心跳连接，就可以使用Service来实现。你可能又会问，前面不是刚刚验证过Service是运行在主线程里的么？在这里一直执行着心跳连接，难道就不会阻塞主线程的运行吗？当然会，但是我们可以在Service中再创建一个子线程，然后在这里去处理耗时逻辑就没问题了。

既然在Service里也要创建一个子线程，那为什么不直接在Activity里创建呢？**这是因为Activity很难对Thread进行控制，当Activity被销毁之后，就没有任何其它的办法可以再重新获取到之前创建的子线程的实例。**而且在一个Activity中创建的子线程，另一个Activity无法对其进行操作。但是**Service就不同了，所有的Activity都可以与Service进行关联，然后可以很方便地操作其中的方法**，即使Activity被销毁了，之后只要重新与Service建立关联，就又能够获取到原有的Service中Binder的实例。因此，使用Service来处理后台任务，Activity就可以放心地finish，完全不需要担心无法对后台任务进行控制的情况。