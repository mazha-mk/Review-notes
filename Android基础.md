# Activity生命周期

![](http://upload-images.jianshu.io/upload_images/3985563-a2caa08d3cbca003.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 

 

1. onCreate()：当 Activity 第一次创建时会被调用。在这个方法中，可以做初始化工作，也可以Bundle对象来回复异常情况下Activity结束时的状态。

 

2. onRestart()：表示Activity正在重新启动。一般情况下，Activity从不可见重新变为可见状态时，onRestart就会被调用。比如用户按Home键切换到桌面或打开了另一个新的Activity，接着用户又回到了这个Actvity。

 

3. onStart(): 表示Activity正在被启动，这时Activity已经出现在后台了，但是还没有出现在前台。这个时候可以理解为Activity已经显示出来，但是我们还看不到。

 

4. onResume():表示Activity已经可见了，并且出现在前台并开始活动。

 

5. onPause():表示 Activity正在停止，仍可见，正常情况下，紧接着onStop就会被调用。onPause中不能进行耗时操作，会影响到新Activity的显示。因为onPause必须执行完，新的Activity的onResume才会执行。

 

6. onStop():表示Activity即将停止，不可见，位于后台。

 

7. onDestory():表示Activity即将销毁，这是Activity生命周期的最后一个回调，可以做一些回收工作和最终的资源回收。

 

### 生命周期的几种情况

1.  按back键回退时，回调如下：

   ```
   onPause()->onStop()->onDestory()
   ```

   

2.  按Home键切换到桌面后又回到该Actitivy，回调如下：onPause()->onStop()->onRestart()->onStart()->onResume()

3.  横竖屏切换：两个回调：

   ```
   onSaveInstanceState()和onRestoreInstanceState()。
   ```

   

 

![](http://upload-images.jianshu.io/upload_images/3985563-23d90471fa7f12d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

OnSaveInstanceState在onStop之前调用。

OnRestoreInstanceState在onStart之后调用。

**其中****onCreate****和****onRestoreInstanceState****方法来恢复****Activity****的状态的区别：** onRestoreInstanceState回调则表明其中Bundle对象非空，不用加非空判断。onCreate需要非空判断。建议使用onRestoreInstanceState。

 

避免横竖屏切换时，Activity的销毁，可以在AndroidManifest文件的Activity设置：

```
android:configChanges = "orientation| screenSize"
（回调方法）
public void onConfigurationChanged(Configuration newConfig){}
```

1.keyboardHidden  :键盘的可访问性发生了改变，比如调出键盘。

2.locale :设备的本地位置发生了改变，比如切换了系统语言。

### Activity的三种运行状态

(1) 前台Activity，优先级最高。

(2) 可见但非前台Activity—比如Activity中弹出了一个对话框，导致Activity可见但是位于后台无法和用户交互。

(3) 后台Activity—比如执行了onStop，优先级最低。

 

### Activity的启动模式

**1.** **标准模式（standard）**

**2.** **栈顶复用模式（singleTop）**

**3.** **栈内复用模式（singleTask）**

**4.** **单例模式（singleInstance）**

 

Activity的管理是采用任务栈的形式，任务栈采用“后进先出”的栈结构。

 

**1.**    **标准模式（standard）：**

每启动一次Activity，就会创建一个新的Activity实例，压入启动它的Activity所在的栈里。

![http://upload-images.jianshu.io/upload_images/3985563-59e0c33aadd55f62.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240](file:///C:/Users/马/AppData/Local/Temp/msohtmlclip1/01/clip_image006.jpg)

特殊情况，如果在Service或Application中启动一个Activity，其并没有所谓的任务栈，可以使用标记位Flag来解决。解决办法：为待启动的Activity指定FLAG_ACTIVITY_NEW_TASK标记位，创建一个新栈。

 

**2.** **栈顶复用模式（singleTop）：**

如果需要新建的Activity位于任务栈栈顶，那么此Activity的实例就不会重建，而是重用栈顶的实例。回调：protected  void onNewIntent(Intent intent){}

由于不会重建一个Activity实例，则不会回调其他生命周期方法。

 

**3.栈内复用模式（singleTask）：**

该模式是一种单例模式，即一个栈内只有一个该Activity实例。

```
可以指定想要加载的目标栈，在AndroidManifest文件的Activity中添加
android:launchMode="singleTask"   android:taskAffinity="com.lvr.task"
```

如果Activity的taskAffinity值没有显式指明，则由Application指明，如果Application没有指明，则默认值为包名，

 

执行逻辑：Activity指定的栈不存在，则创建一个栈；Activity指定的栈存在，其中没有该Activity实例，则会创建Activity并压入栈顶；其中有该Activity实例，则把该Activity实例之上的Activity杀死清除出栈，重用并让该Activity实例处在栈顶，然后调用onNewIntent()方法。（**singleTask****会具有****clearTop****特性**）

 

场景：第一次进入主界面之后，主界面位于栈底，以后不管我们打开了多少个Activity，用singleTask让主界面Activity处于栈顶，保证退出应用时所有的Activity都能报销毁。

 

**4.** **单例模式（singleInstance）：**

打开该Activity时，直接创建一个新的任务栈，并创建该Activity实例放入新栈中；任何应用再激活该Activity时都会重用该栈中的实例。

 

**Activity的Flags**

可以在启动Activity时，通过Intent的addFlags()方法设置。

 

**(1)FLAG_ACTIVITY_NEW_TASK** 其效果与指定Activity为singleTask模式一致。

**(2)FLAG_ACTIVITY_SINGLE_TOP** 其效果与指定Activity为singleTop模式一致。

**(3)FLAG_ACTIVITY_CLEAR_TOP** 具有此标记位的Activity，当它启动时，在同一个任务栈中所有位于它上面的Activity都要出栈。如果和singleTask模式一起出现，若被启动的Activity已经存在栈中，则清除其之上的Activity，并调用该Activity的onNewIntent方法。如果被启动的Activity采用standard模式，那么该Activity连同之上的所有Activity出栈，然后创建新的Activity实例并压入栈中。

**IntentFilter的匹配规则：**

（注：一个Activity中可以有多个intent-filter，一个隐式Intent只要能匹配任何一组intent-filter即可成功启动对应的Activity）

**1.action的匹配规则**

一个intent-filter可以有多个action，只要Intent全部的action，至少有一个intent-filter有相同对应的全部action，即匹配成功。

**2.category的匹配规则**

1. Intent如果有category，只要Intent全部的category，至少有一个intent-filter有相同对应的全部category，即匹配成功。

 

2. Intent如果没有category,只要Activity的有一个intent-filter没有设置category的匹配规则，则匹配成功。（实质：没有设置category的匹配规则的intent-filter会默认有一个

   ```
   <category android:name="android.intent.category.DEFAULT"/>）
   ```

   

**3.data的匹配规则**

1. data分为mimeType（媒体类型）和URI两部分。

2. data的匹配规则和action类似。

**4.主Activity的intent-filter**

```
<action android:name="android.intent.action.MAIN" />

<category android:name="android.intent.category.LAUNCHER" />
```

这二者共同作用是用来标明这是一个入口Activity，并且会出现在系统的应用列表中。

# Service

1. 与Activity一样都是Context的子类，只不过它没有UI界面，是在后台运行的组件。

2. 执行在UI线程中，Service的耗时操作需要创建子线程完成。

   ----

### **按运行地点种类**

**1.**     **本地服务（Local Service）：**

特点：依附在主进程上，不是独立的进程，生命随着主进程终止而终止。

**2.**     **远程服务（Remote Service）:**

特点：独立的进程进程名格式为包名加上android:process属性制定的字符串。需要使用AIDL进行IPC。

 

### **生命周期**

![](http://upload-images.jianshu.io/upload_images/3985563-85614addbbec7a0c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. OnCreate()：**第一次创建时执行且**只运行一次的**初始化工作

 

2.  onStartCommand()：每次客户端调用startService()方法启动该Service都会回调该方法（多次调用）。通过调用stopSelf()或stopService()来停止服务。

 

3.  OnBind()：当组件调用bindService()想要绑定到service时系统调用此方法（一次调用，一旦绑定后，下次再调用bindService()不会回调该方法）。

4. OnUnbind()：组件调用unbindService()，想要解除与service的绑定时系统调用此方法（一次调用，一旦解除绑定后，下次再调用unbindService()会抛出异常）。

5. OnDestory()：在service不再被使用并要销毁时调用此方法（一次调用）。

 

#### **startService / stopService**

生命周期顺序：onCreate(一次)->onStartCommand(多次)->onDestroy(一次)

 

***注意点：***

1. 不论 startService 多少次，stopService 一次就会停止服务

2. 一个service被Context.startService启动，不论Activity使用bindService或者unbindService，该Service都在后台运行，直到被调用stopService，或自身的stopSelf方法，才结束服务。

 

#### **bindService / unbindService**

生命周期顺序：onCreate(一次)->onBind(一次)->onUnBind(一次)->onDestroy多次)

 

***注意点：***

1. 第一次 bindService 会触发 onCreate 和 onBind，以后在服务运行过程中，每bindService 都不会触发任何回调。

 

####  **混合型（上面两种方式的交互）**

***注意点：***

1. 不管startService和bindService调用多少次，onCreate也只会执行一次。

2. 调用unBindService将不会停止Service，必须调用stopService或Service自身的stopSelf来停止服务。

 

**注意与配置：**

```
1.配置远程服务，在配置文件的service里配置android:process=”remote”属性
```

2. 要在onDestroy()中进行有必要的清理工作。

3. **不可交互的服务**，直接startService

4. **可交互的服务**，使用bindService，通过bindService启动的Service的生命周期依附于启动它的Context。因此当前台调用bindService的Context销毁后，那么服务会自动停止。(后台服务的情况下)

5. **前台服务**，即是在Service的基础上创建一个Notification，然后使用Service的startForeground()方法，即可启动为前台服务。

​       使用TaskStackBuilder实现点击Notification跳转到指定界面后，按back键，返回到软件的主界面。

```
1.TaskStackBuilder.create(this)，创建一个stackBuilder实例。
2. stackBuilder.addParentStack(NotificationShow.class);
3.    还要在manifest文件设置：
<activity
    android:name="com.lvr.service.NotificationShow" 
    android:parentActivityName=".MainActivity" >
</activity>
```

~~~java
mBuilder = new NotificationCompat.Builder(this)
        .setContent(view)
        .setSmallIcon(R.drawable.icon).setTicker("新资讯")
        .setWhen(System.currentTimeMillis())
        .setOngoing(false)
        .setAutoCancel(true);
Intent intent = new Intent(this, NotificationShow.class);
TaskStackBuilder stackBuilder = TaskStackBuilder.create(this);
stackBuilder.addParentStack(NotificationShow.class);
stackBuilder.addNextIntent(intent);
PendingIntent pendingIntent = stackBuilder.getPendingIntent(0, PendingIntent.FLAG_UPDATE_CURRENT);
//PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT);
mBuilder.setContentIntent(pendingIntent);
~~~



6. 通过**stopForeground()****方法可以取消通知，即将前台服务降为后台服务。此时服务依然没有停止。通过stopService()可以把前台服务停止。

 

 

# BroadcastReceiver

作用：

1. 用于监听/接收应用发出的广播消息，左右响应。

2. 不同组件之间的通信。

3. 与android系统在特定情况下通信。

4. 多线程通信。

----

原理：设计模式中的**观察者模式**：基于消息的发布/订阅事件模型。

模型中有3个角色：

1. 消息订阅者(广播接收者)

2. 消息发布者(广播发布者)

3. 消息中心（AMS，即Activity Manager Service）

描述：

1. 广播接收者/广播发送者通过Binder机制向AMS注册/发送广播

2. AMS通过IntentFilter/Permission寻找合适的接收者。

3. 广播接收者通过消息循环拿到广播，回调onReceive().

 ### 注册

#### 1.静态注册

在AndroidManifest.xml里通过 标签声明。

~~~xml
<receiver       
  //继承BroadcastReceiver子类的类名
  android:name=".mBroadcastReceiver"
          
  //具有相应权限的广播发送者发送的广播才能被此BroadcastReceiver所接收；
  android:permission="string"
          
  //BroadcastReceiver运行所处的进程
  //默认为app的进程，可以指定独立的进程
  //注：Android四大基本组件都可以通过此属性指定自己的独立进程
  android:process="string" >

  //用于指定此广播接收器将接收的广播类型
  //本示例中给出的是用于接收网络状态改变时发出的广播
  <intent-filter>
    <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
  </intent-filter>
</receiver>
~~~

#### 2.动态注册

在代码中通过Context的registerReceiver()方法进行动态注册BroadcastReceiver。

~~~java
//实例化BroadcastReceiver子类 &  IntentFilter
    mBroadcastReceiver mBroadcastReceiver = new mBroadcastReceiver();
    IntentFilter intentFilter = new IntentFilter();

    //设置接收广播的类型
    intentFilter.addAction(android.net.conn.CONNECTIVITY_CHANGE);

    //调用Context的registerReceiver（）方法进行动态注册
    registerReceiver(mBroadcastReceiver, intentFilter);

-------------在onDestroy()中注册接收者-------------
  //销毁在onResume()方法中的广播
    unregisterReceiver(mBroadcastReceiver);  
~~~

***注意***

1. 对于动态广播，有注册就必然得有注销，否则会导致**内存泄露**

> 重复注册、重复注销也不允许

#### 区别

![静态注册与动态注册的区别](http://upload-images.jianshu.io/upload_images/944365-0ae738c6d50c0adf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240 "区别")

 ### 广播

#### 广播发送

广播发送者将此广播的”意图“通过sendBroadcast(Intent)方法进行发送广播。

#### 广播的类型

1. 普通广播

> 即开发者自身定义intent的广播（最常用）
>
> >通过使用action与注册的广播接收者是IntentFilter内的action想匹配的Intent，然后sendBroadcast(Intent)来实现发送。

2. 系统广播

> Android内置了多种系统广播，每个广播都有相应的Intent-Filter。
>
> >只需在注册广播接收者时定义相关的action即可，系统有相关操作时会自动进行系统广播。

3. 有序广播

> 广播会被广播接收着按先后顺序接收：
>
> > 按Priority属性值从大到小排序，属性值相同的动态注册优先。
>
> 先接收的接收者可以进行截断，也可以进行修改。

> `sendOrderedBroadcast(intent);`

4. App应用内广播

 Android中的广播可以跨App直接通信（exported对于有intent-filter情况下默认值为true）

> 使用方法
>
> > 1. 注册广播接收者时将`android:exported = false` ，即非本App内部发出的广播不被接收。
> > 2. 发送广播时指定该广播接收器所在的包名。通过`intent.setPackage(packageName) ` 指定包名。

5. 粘性广播

> 在Android5.0失效。
>
> 即是指广播接收器一注册马上就能接收到广播的一种机制，当然首先系统要存在广播。

# ContentProvider

### 作用

可以用于进程间的通信。

### 原理

使用binder机制

### 注册

~~~xml
<provider 
   //ContentProvider的类的路径
   android:name="MyProvider"
   //唯一标识符
     android:authorities="唯一标识符"
   // 设置此provider是否可以被其他进程使用
     android:exported="true"        
          
          />
~~~



### 使用

![img](http://upload-images.jianshu.io/upload_images/944365-5c9b0e2ebed36c3f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 1. 统一资源标识符 (URI)

外界进程通过 URI 找到对应的ContentProvider & 其中的数据，再进行数据操作。

![img](http://upload-images.jianshu.io/upload_images/944365-96019a2054eb27cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2.MIME数据类型

#### 3.ContentProvider类

1. 组织数据方式：以表格的形式组织数据。

2. 主要的方法有：增，删，查，改

> ```四个方法运行在binder线程池中(不是主线程)
> 
> ```

~~~java
<-- 4个核心方法 -->
  public Uri insert(Uri uri, ContentValues values) 
  // 外部进程向 ContentProvider 中添加数据

  public int delete(Uri uri, String selection, String[] selectionArgs) 
  // 外部进程 删除 ContentProvider 中的数据

  public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs)
  // 外部进程更新 ContentProvider 中的数据

  public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs,  String sortOrder)　 
  // 外部应用 获取 ContentProvider 中的数据

~~~

3. 其他方法：

~~~java
<-- 2个其他方法 -->
public boolean onCreate() 
// ContentProvider创建后 或 打开系统后其它进程第一次访问该ContentProvider时 由系统进行调用
// 注：运行在ContentProvider进程的主线程，故不能做耗时操作

public String getType(Uri uri)
// 得到数据类型，即返回当前 Url 所代表数据的MIME类型
~~~

**使用：**

* Android有很多常见的数据(如通讯录、日程表等)的ContentProvider可以使用。
* 自定义ContentProvider需要实现上面的六个方法。
* ContentProvider类并不会直接与外部进程交互，而是通过ContentResolver 类。

#### 4.ContentResolver类

**作用**

外部进程通过 `ContentResolver`类 从而与多个`ContentProvider`类进行交互。

**使用**

`ContentResolver` 类提供了与`ContentProvider`类相同名字 & 作用的4个方法。

使用例子：

~~~java
// 使用ContentResolver前，需要先获取ContentResolver
ContentResolver resolver =  getContentResolver(); 

// 设置ContentProvider的URI
Uri uri = Uri.parse("content://cn.scu.myprovider/user"); 

// 根据URI 操作 ContentProvider中的数据
// 此处是获取ContentProvider中 user表的所有记录 
Cursor cursor = resolver.query(uri, null, null, null, "userid desc");
~~~

#### 5. 辅助类

1. ContentUris类

> 作用：操作 `URI`,使用两个方法：`withAppendedId（）` &`parseId（）`

2. UriMatcher类

> 根据URI匹配ContentProvider

3. ContentValues类

> 装载数据

4. Cursor类

#### 6.ContentObserver类

1. 作用:观察 Uri引起ContentProvider 中的数据变化 & 通知外界（即访问该数据访问者）

2. 使用：

~~~java
// 步骤1：注册内容观察者ContentObserver
getContentResolver().registerContentObserver（uri）；
// 通过ContentResolver类进行注册，并指定需要观察的URI

// 步骤2：当该URI的ContentProvider数据发生变化时，通知外界（即访问该ContentProvider数据的访问者）
public class UserContentProvider extends ContentProvider {
    public Uri insert(Uri uri, ContentValues values) {
        db.insert("user", "userid", values);
        getContext().getContentResolver().notifyChange(uri, null);
        // 通知访问者
    }
}

// 步骤3：解除观察者
getContentResolver().unregisterContentObserver（uri）；
// 同样需要通过ContentResolver类进行解除
~~~

### 优点

1. 安全
2. 访问简单&高效





#  Fragment

### 生命周期

因为Fragment是依附于Activity存在的，因此它的生命周期收到Activity的生命周期影响。

![img](http://upload-images.jianshu.io/upload_images/1780352-f8584bc70f3c149c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* onAttach():当Fragment与Activity发生关联时调用。
* onCreateView(LayoutInflater,ViewGroup,Bundle):创建Fragment的视图。
* onActivityCreated(Bundle):当Activity的onCreate()返回时调用。
* onDestroyView():当Fragment的视图被移除调用。
* onDetach():在Fragment与Activity取消关联时调用。

PS：注意：除了onCreateView，其他的所有方法如果你重写了，必须调用父类对于该方法的实现。即super();

### Fragment的使用

#### 1.静态使用Fragment

1. 创建一个类继承Fragment，重写onCreateView方法，来确定Fragment要显示的布局。
2. 在Activity的xml中声明该类，与普通的View对象一样。

~~~xml
 <fragment
       android:id="@+id/myfragment"
      //指明fragment的类的包名
       android:name="com.usher.fragment.MyFragment"
       android:layout_width="match_parent"
       android:layout_height="match_parent" />
~~~

#### 2. 动态使用Fragment

1. 在Acitivity对应的布局中写上一个**FrameLayout**控件，此空间的作用是当作Fragment的容器，**Fragment**通过**FrameLayout**显示在Acitivity里

~~~xml
 <FrameLayout
        android:id="@+id/myframelayout"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

~~~

2. 实例化好你的Fragment类。
3. 通过getSupportFragmentManager()方法新建FragmentManager对象。
4. 使用FragmentManager的进行**事务**。

**事务方法**

* **beginTransaction()**开启一个事务。

- **transaction.add()** 向Activity中添加一个Fragment。
- **transaction.remove()** 从Activity中移除一个Fragment，如果被移除的Fragment没有添加到**回退栈**，这个Fragment实例将会被销毁。
- **transaction.replace()** 使用另一个Fragment替换当前的，实际上就是**remove()+add()的合体**。
- **transaction.hide()** 隐藏当前的Fragment，仅仅是设为不可见，并不会销毁。
- **transaction.show()** 显示之前隐藏的Fragment。
- **detach()** 会将view从UI中移除,和remove()不同,此时fragment的状态依然由FragmentManager维护。
- **attach()** 重建view视图，附加到UI上并显示。
- **ransatcion.commit()** 提交事务。

**回退栈**

保存每一次Fragment事务发生的变化，如果你将Fragment任务添加到回退栈，当用户点击后退按钮时，将看到上一次的保存的Fragment。

**FragmentTransaction.addToBackStack(String)**{把当前事务的变化情况添加到回退栈}



### Fragment与Activity之间的通信

1. Activity包含自己管理的Fragment，则直接通过引用Fragment的public方法通信。
2. Activity不包含自己管理的Fragment，每个Fragment都有唯一的TAG/ID，可通过getFragmentManager.findFragmentByTag()或者findFragmentById()获得任何Fragment实例，然后通信。
3. Fragment中可以通过getActivity()得到当前绑定的Activity的实例，然后进行操作。

### 优化

1. 降低Fragment与Activity的耦合，可以使用接口。
2. Fragment更不应该直接操作别的Fragment。



# 消息机制概述

Handler是Android消息机制的上层接口。在子线程中，进行耗时操作，执行完操作后，发送消息，通知主线程更新UI。这就是Handler的用法。

解决在子线程无法访问UI的矛盾。

### 模型

主要包括：**MessageQueue**,**Handler**,**Looper**,**Message**。

* Message: 需要传递的消息。

* MessageQueue：消息队列，实际上是通过一个单链表的数据结构来维护消息列表。主要功能向消息池投递消息(MessageQueue.enqueueMessage)和取走消息池的消息(MessageQueue.next)。

* Handler：消息辅助类，主要功能向消息池发送各种消息事件(Handler.sendMessage)和处理相应消息事件(Handler.handleMessage)。

* Looper：不断循环执行(Looper.loop)，从MessageQueue中读取消息，按分发机制将消息分发给目标处理者。

  > 每个线程中只能存在一个Looper，Looper是保存在ThreadLocal中的。

### 架构

![img](http://upload-images.jianshu.io/upload_images/3985563-d7da4f5ba49f6887.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

Looper有一个MessageQueue消息队列；MessageQueue有一组待处理的Message；Message中记录发送和处理消息的Handler；
Handler中有Looper和MessageQueue。

**Looper**

无限循环获取MessageQueue里的Message，然后将Message分发到目标Handler。

使用Looper.prepare()即可为当前的线程创建一个Looper

通过Looper.loop()来开启消息循环

可通过Looper.quit()/quitSafely()退出消息循环，前者直接退出，后者会把队列内消息处理后再退出。

**Handler**

当使用Handler发送Message时，即是将Message使用MessageQueue.enqueueMessage向消息池投递消息.

Handler的Message，是Looper分发出来的，Looper会调用：

1. 当Message的`msg.callback`不为空时，则回调方法`msg.callback.run()`;优先级最高
2. 当Handler的`mCallback`不为空时，则回调方法`mCallback.handleMessage(msg)`；优先级仅次于1；
3. 最后调用Handler自身的回调方法`handleMessage()`，该方法默认为空，Handler子类通过覆写该方法来完成具体的逻辑。优先级最低。

### ThreadLocal类

* 是一个线程内部的数据存储类。
* 每一个线程Thread类有一个专门用于存储ThreadLocal的数据对象
* 对一个ThreadLocal使用get/put操作，内部会获取当前线程内部存储的数据对象，进行操作。

### 总结

![img](http://upload-images.jianshu.io/upload_images/3985563-b3295b67a2b0477f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

# AsyncTask

### AsyncTask简介

* AsyncTask是一个抽象类，由Android封装的轻量级异步类(使用方便，代码简洁)。
* AsyncTask内部封装两个线程池(SerialExecutor与THREAD-POOL_EXECUTOR)和一个Handler(InternalHandler)。
* SerialExecutor线程池用于任务的排队；THREAD_POOL_EXECUTOR线程池才真正用于执行任务；InternalHandler用于从工作线程切换到主线程。

### AsyncTask抽象类

~~~java
public abstract class AsyncTask<Params, Progress, Result>
~~~

泛型类型参数得含义：

**Params:**开始异步任务执行时传入的参数类型

**Progress:**异步任务执行过程中，返回下载进度值得类型

**Result:**异步任务执行完成后，返回得结果类型

**如果AsyncTask确定不需要传递具体参数，那么这三个泛型参数可以用Void来代替。**

### AsyncTack核心方法

**onPreExecute()**

> 后台任务开始执行之前调用，一般用于一些初始化操作
>
> 执行线程：主线程

**Result doInBackground(Params...)**

> 在这里处理所有的耗时任务
>
> 如果需要更新UI元素，可以调用**publishProgress(Progress...)方法完成**
>
> 执行线程：子线程

**onProgressUpdate(Progress...)**

> 在publishProgress(Progress...)被调用后，很快被调用，可利用参数进行UI的更新
>
> 执行线程：主线程

**onPostExecute(Result)**

> 当doInBackground(Params...)执行完return返回时，这个方法很快被调用，返回的数据会作用此方法的参数。
>
> 执行线程：主线程

**onCanelled()**

> 当异步任务被AsyncTask.cancel()取消时会被调用，onPostExecute(R)不会被调用
>
> 执行线程：主线程
>
> **AsyncTask中的cancel()方法并不是真正去取消任务，只是设置这个任务为取消状态，我们需要在doInBackground()判断终止任务。就好比想要终止一个线程，调用interrupt()方法，只是进行标记为中断，需要在线程内部进行标记判断然后中断线程。**

### 注意事项

1. 异步任务的实例必须在UI线程中创建，即AsyncTask对象必须在UI线程中创建。

2. execute(Params... params)方法必须在UI线程中调用。

3. 不要手动调用onPreExecute()，doInBackground(Params... params)，onProgressUpdate(Progress... values)，onPostExecute(Result result)这几个方法。

4. 一个AsyncTask实例只能执行一次，如果执行第二次将会抛出异常。

### AsyncTask的串行和并行

AsyncTask是串行运行的

要想并行运行，直接采用AsyncTask的executeOnExecutor方法即可。

### AsyncTask使用不当

1. 生命周期

AsyncTask不与任何组件绑定生命周期，最好要在Activity/Fragment的onDestroy()调用cancel(boolean)

2. 内存泄漏

AsyncTask被声明为Activity的非静态内部类，AsyncTask会保留创建其的Activity的引用。如果Activity已经被销毁，而AsyncTask仍在运行，则会导致Activity无法被回收，引起内存泄露

3. 结果丢失

当屏幕旋转或者Activity在后台被系统杀掉等情况会导致Activity的重新创建，之前运行的AsyncTask（非静态的内部类）会持有一个之前Activity的引用，这个引用已经无效，这时调用onPostExecute()再去更新界面将不再生效。

# HandlerThread

HandlerThread是一个内部创建了消息队列的Thread类。

HandlerThread的run()方法，不想其它的Thread用于执行耗时任务，而是创建Loop，然后开启消息的无限循环。

~~~java
@Override
    public void run() {
        mTid = Process.myTid();
        Looper.prepare();
      	...
        Looper.loop();
        mTid = -1;
    }
~~~

### 使用

~~~java
 //创建一个线程,线程名字：handler-thread
HandlerThread ht = new HandlerThread( "handler-thread") ;
//开启一个线程
ht.start();
//在这个线程中创建一个handler对象
//handler的loop使用HandlerThread的，以使handleMessage执行在handler-thread 线程中
handler = new Handler( ht.getLooper() ){
    //这个方法是运行在 handler-thread 线程中的,可以执行耗时任务
    @Override
    public void handleMessage(Message msg) {
            super.handleMessage(msg);
            //执行耗时任务
            }
        };

 //通知HandlerThread执行任务
handler.sendEmptyMessage( 1 ) ;
~~~

### 注意

由于HandlerThread的run()是一个无限循环，因此，在不需要使用是要通过HandlerThread的quit()/quickSafely()方法来终止线程的执行。



# IntentService

IntentService是一个继承了Service的抽象类，封装了HandlerThread和Handler。

IntentService可用于执行后台耗时的任务，当任务执行完后他会自动停止。

### 使用

1. 定义IntentService的子类：传入线程名称、复写onHandleIntent()方法

~~~java
public class myIntentService extends IntentService {
....
    /*复写onHandleIntent()方法*/
    //实现耗时任务的操作
    @Override
    protected void onHandleIntent(Intent intent) {
        //根据Intent的不同进行不同的事务处理
        String taskName = intent.getExtras().getString("taskName");
        switch (taskName) {
            case "task1":
                Log.i("myIntentService", "do task1");
                break;
            case "task2":
                Log.i("myIntentService", "do task2");
                break;
            default:
                break;
        }
    }
  ....
}
~~~

2. 在Manifest.xml中注册IntentService
3. 在Activity中开启Service服务并使用

~~~java
Intent i = new Intent("cn.scu.finch");
Bundle bundle = new Bundle();
bundle.putString("taskName", "task1");
i.putExtras(bundle);
startService(i);
~~~

### 原理

IntentService内部封装了HandlerThread和Handler，Handler的Looper使用HandlerThread的，并且Handler的handleMessage(Message)方法内部调用抽象方法onNewIntent(Intent intent)得以实现。



# Android动画

#### Android动画的种类 ：

* 视图动画
* 帧动画
* 属性动画
* 关键帧动画
* 触摸反馈动画（Ripple Effect）
* 揭露动画（Reveal Effect）
* 转场动画&共享元素
* 视图状态动画（Animate View State Changes）
* 矢量图动画（Vector 动画）
* 其他动画

#### 视图(补间)动画

##### 类型

|    名称    |    标签     |       JAVA类       |
| :--------: | :---------: | :----------------: |
|  平移动画  | <translate> | TranslateAnimation |
|  缩放动画  |   <scale>   |   ScaleAnimation   |
|  旋转动画  |  <rotate>   |  RotateAnimation   |
| 透明度动画 |   <Alpha>   |   AlphaAnimation   |
|  动画集合  |    <Set>    |    AnimationSet    |

(xml要新建res/anim/xx.xml路径来存放)

##### 属性

~~~xml
<set android:interpolatior=".." //插值器
     android:shareInterpolator="boolean" //是否共享同一插值器
     android:duration=""
     android:fillAfter=""//结束后是否停留在结束位置
     >
    <translate android:fromXDelta=""//x的起始值
               android:fromYDelta=""//y的起始值
               android:toXDelta=""//x的结束值
               android:toYDelta=""//y的结束值
               />
    <scale android:fromXScale="0.5"//水平方向的起始值
           android:toXScale="1.2"//水平方向的结束值
           android:fromYScale=""//竖直方向的起始值
           android:toYScale=""//竖直方向的结束值
           android:pivotX=""//缩放的轴点的x坐标
           android:pivotY=""//缩放的轴点的y坐标
           
           />
    <rotate android:fromDegrees="0"//旋转开始的角度
            android:toDegrees="180"//旋转结束的角度
            android:pivotX=""//旋转的轴点的x坐标
            android:pivotY=""//旋转的轴点的y坐标
            />
    <alpha android:fromAlpha=""//透明度的起始值
           android:toAlpha=""//透明度的结束值，[0,1]
           />
</set>
~~~

（java类有对应的属性）

##### 监听

~~~java
public static interface AnimationListener{
    void onAnimatonStart(Animation animation);
     void onAnimatonEnd(Animation animation);
     void onAnimatonRepeat(Animation animation);
}
~~~



### 帧动画

按顺序播放一组预先定义好的图片。

使用<animation-list>与AnimationDrawable实现。

~~~xml
<animation-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:drawable="@drawable/0" android:duration="100" />
    <item
        android:drawable="@drawable/1" android:duration="100" />
    <item
        android:drawable="@drawable/2" android:duration="100" />
</animation-list>
~~~

~~~java
ImageView iv = (ImageView) findViewById(R.id.animation1);
iv.setImageResource(R.drawable.frame_anim1);
AnimationDrawable animationDrawable1 = (AnimationDrawable) iv.getDrawable();
animationDrawable1.start();
~~~

##### 注意

容易造成OOM

### 属性动画

##### 类型

|       标签       |     JAVA类     |
| :--------------: | :------------: |
|    <animator>    | ValueAnimator  |
| <objectAnimator> | ObjectAnimator |
|      <Set>       |  AnimatorSet   |

（(xml要新建res/animator/xx.xml路径来存放）

##### 属性

~~~xml
<set android:ordering="together|sequentially"
     //设置子动画同时播放|逐个播放>
	<objectAnimator android:propertyName="string"
                    android:duration=""
                    android:valueFrom=""
                    android:valueTo=""
                    android:startOffset="int"//动画延迟时间
                    android:repeatCount=""
                    android:repeatMode="restart|reverse"
                    android:valueType="intType|floatType"
                    />
	<animator //属性与objectAnimator基本相同>
</set>
~~~

```java
AnimatorSet set = (AnimatorSet)AnimatorInflater.loadAnimator(Context,R.animator.xx);
set.setTarget(view);
set.start();
```

（java类有对应的属性）

~~~
objectAnimator.ofInt(view,"属性名-驼峰",int...value)
~~~

**ObjectAnimator内部时通过View相对应的属性的get/set方法实现的，可通过此原理实现自定义属性的动画。**

##### 监听

~~~java
public static interface AnimatorListener{
	void onAnimatorStart(Animator animator);
    void onAnimatorEnd(Animator animator);
    void onAnimatorCancel(Animator animator);
    void onAnimatorRepeat(Animator animator);
}
~~~

~~~java
public static interface AnimatorUpdateListener{
	void onAnimationUpdate(ValueAnimator animator);
    //用来配合进度值更新动画的view的属性值
}
~~~

##### 插值器

~~~java
public interface TimeInterpolator{
	public abstract float getInterpolation (float input)；//时间进度[0,1]
        //需返回在input时间进度时，动画的进度值
}
~~~



##### 估值器

~~~java
public interface TypeEvaluator<T>{
    //fraction:估值小数，即Interpolator返回的进度值
    //startValue:开始值
    //endValue:结束值
    //需返回当前进度的属性值
    public abstract T evaluate (float fraction, 
                T startValue, 
                T endValue);
} 
~~~

##### 注意

如果在Activity退出时没有及时将动画停止，属性动画会导致Activity无法释放而导致内存泄漏。(使用属性动画时切记在Activity执行 onStop 方法时顺便将动画停止。)

### 关键帧动画

### 触摸反馈动画

### 揭露动画

### 转场动画

### 视图状态动画

### 矢量图动画

#### AnimatedVectorDrawable

 AnimatedVectorDrawable通过`ObjectAnimator`或`AnimatorSet`对`VectorDrawable`的某个属性作动画。

从API-25开始，AnimatedVectorDrawable运行在RenderThread(相反地，早期API是运行在UI线程)。这也就是说AnimatedVectorDrawable在UI线程繁忙时也能保持流畅运行。

#### 标签

**<vector>标签：**定义一个矢量图。

**<path>标签：**定义一个路径。

`<path>的pathData属性`可用的命令如下：

~~~
M = moveto(M X,Y) ：                                                    将画笔移动到指定的坐标位置
L = lineto(L X,Y) ：                                                    画直线到指定的坐标位置
H = horizontal lineto(H X)：                                            画水平线到指定的X坐标位置
V = vertical lineto(V Y)：                                              画垂直线到指定的Y坐标位置
C = curveto(C X1,Y1,X2,Y2,ENDX,ENDY)：                                  三次贝赛曲线
S = smooth curveto(S X2,Y2,ENDX,ENDY)                                   同样三次贝塞尔曲线，更平滑
Q = quadratic Belzier curve(Q X,Y,ENDX,ENDY)：                          二次贝赛曲线
T = smooth quadratic Belzier curveto(T ENDX,ENDY)：                     同样二次贝塞尔曲线，更平滑
A = elliptical Arc(A RX,RY,XROTATION,large-arc-flag,sweep-flag,X,Y)：   弧线
Z = closepath()：                                                       关闭路径

~~~

**<group>标签**：定义用来设置路径做动画的属性。

**<clip-path>标签**:定义当前绘制的剪切路径。

**<animated-vector>标签**：关联矢量图与动画。

**<set>,<objectAnimator>标签**：属性动画。

#### Android SVG 动画

**一.新建三个XML文件**

1. 在res/drawable/中添加定义矢量图的XML，根标签为<vector>

~~~xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="96dp"
    android:height="96dp"
    android:viewportWidth="96.0"
    android:viewportHeight="96.0">
        <group android:name="rotationGroup">
            <path
                android:fillColor="@color/colorPrimary"
                android:pathData="M48 48 l68 96"/>
        </group>
</vector>
~~~

2. 在res/anim/中添加定义矢量图的动画XML，根标签为动画根标签<set>,<objectAnimator>

~~~xml
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <objectAnimator
        android:duration="3000"
        android:propertyName="pathData"
        android:valueFrom="M48 48 88 96"
        android:valueTo="M48 80 88 96"
        android:valueType="pathType"/>
</set>
~~~



3. 在res/drawable/中添加矢量图与动画进行关联的XML，根标签为<animated-vector>

~~~xml
<animated-vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/vectordrawable"//1文件 >
    <target
        android:name="rotationGroup"
        android:animation="@anim/rotation"//2文件 />
</animated-vector>

~~~

**二.在代码上运用**

~~~java
ImageView iv = ...;
iv.setImageResource(R.drawable.avd_cry);
Drawable drawable = iv.getDrawable();
if (drawable instanceof Animatable)
    ((Animatable) drawable).start();

~~~

或者：

~~~xml
<ImageView xmlns:app="http://schemas.android.com/apk/res-auto"
    ...
       //使用srcCompat兼容5.0以下版本
    app:srcCompat="@drawable/animated_v"/>
~~~

~~~java
ImageView iv= (ImageView) findViewById(R.id.imageview);
((AnimatedVectorDrawableCompat)iv.getDrawable()).start();
~~~



### 其他动画

# Context

#### 了解

Context提供了**关于应用环境全局信息的接口**。它是一个抽象类，它的执行被Android系统所提供。它允许获取以应用为特征的资源和类型，是一个统领一些资源（应用程序环境变量等）的上下文。就是说，它描述一个应用程序环境的信息（即上下文）。

![img](http://upload-images.jianshu.io/upload_images/1187237-1b4c0cd31fd0193f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Context有两个具体的实现子类：ContextImpl和ContextWrapper。

ContextWrapper这是一个包装类，内部持有一个ContextImpl引用，调用ContextWrapper的方法都会被转向其所包含的真正的Context对象。

### 数量

**一个应用程序的Context数量=Activity数量+Service数量+1。**

Broadcast Receiver，Content Provider并不是Context的子类，他们所持有的Context都是其他地方传过去的，所以并不计入Context总数。

### 作用域

![img](http://upload-images.jianshu.io/upload_images/1187237-fb32b0f992da4781.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意：

1. 与UI有关的使用Activity作为Context最为合适。

2. 其他操作，Application,Activity,Service作为Context都可以。
3. 小心Context的持有，容易内存泄漏。

**getApplication()和getApplicationContext()的区别：**

1. 同时返回的同一个对象
2. getApplication()只有在Activity和Service才能调用到
3. getApplicationContext()可以在除Activity和Service外，比如BroadcastReceiver中使用。

### Context的使用

1. 当Application的Context能搞定的情况下，并且生命周期长的对象，优先使用Application的Context。
2. 不要让生命周期长于Activity的对象持有到Activity的引用。
3. 尽量不要在Activity中使用非静态内部类，因为非静态内部类会隐式持有外部类实例的引用，如果使用静态内部类，将外部实例引用作为弱引用持有。

# Bitmap的高效加载

### 加载Bitmap的方式

1. BitmapFactory.decodeFile:文件系统
2. BitmapFactory.decodeResource:资源文件
3. BitmapFactory.decodeStream:输入流
4. BitmapFactory.decodeByteArray:字节数租

### BitmapFactory.Options的参数

1. inSampleSize参数：采样率

> **缩放比例为1/(inSampleSize^2)**
>
> 例如,4M的图片，inSampleSize=2，则采样后为1M
>
> **inSampleSize应该总是2的指数，否则系统会向下取整选择一个2的指数代替**

2. inJustDecodeBounds参数

> inJustDecodeBounds=true,加载图片只会解析宽高信息，不会真正加载图片，可以在次进行缩放。
>
> inJustDecodeBounds=false,真正加载图片

### 代码实现

~~~java
public static Bitmap decodeSampledBitmapFromResource(Resources res, int resId, int reqWidth, int reqHeight){
        BitmapFactory.Options options = new BitmapFactory.Options();
        options.inJustDecodeBounds = true;
        //加载图片
        BitmapFactory.decodeResource(res,resId,options);
        //计算缩放比
        options.inSampleSize = calculateInSampleSize(options,reqHeight,reqWidth);
        //重新加载图片
        options.inJustDecodeBounds =false;
        return BitmapFactory.decodeResource(res,resId,options);
    }

~~~

### 使用Matrix实现图片缩放

~~~java
//将图片缩放到对应的大小的图片，
    //参数1：原图片，参数2：目标大小
    //方法放回最终图片
    //没测试过
    public static Bitmap  adjustBitmapForSize(Bitmap curBitmap,int destWidth,int destHeight){
        int rawWidth  = curBitmap.getWidth();
        int rawHeight = curBitmap.getHeight();
        float scaleHeight = ((float)destHeight)/rawHeight;
        float scaleWidth = ((float)destWidth)/rawWidth;

        Matrix matrix = new Matrix();
        matrix.postScale(scaleWidth, scaleHeight);

        Bitmap newbitmap = Bitmap.createBitmap(curBitmap, 0, 0, rawWidth, rawHeight, matrix, true);
        return newbitmap;
~~~

