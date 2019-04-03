# MVP

### MVP与MVC

在MVP 架构中跟MVC类似的是同样也分为三层。

**MVC**：

1. **View层**：对应于布局文件

2. **Model 层**：业务逻辑和实体模型

3. **Controllor层**：对应于Activity

![img](http://upload-images.jianshu.io/upload_images/3985563-4472518336d7465e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**MVP**：

1. **View层**：Activity 和Fragment ，负责处理 UI。
2. **Presenter层**：为业务处理层，既能调用UI逻辑，又能请求数据，该层为纯Java类，不涉及任何Android API。
3. **Model 层**：中包含着具体的数据请求，数据源。

![img](http://upload-images.jianshu.io/upload_images/3985563-7c936f2223dce1de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**比较：**MVP模式减少了Activity的职责，简化了Activity中的代码，将复杂的逻辑代码提取到了Presenter中进行处理，模块职责划分明显，层次清晰。与之对应的好处就是，耦合度更低，更方便的进行测试。

### 总结

MVP中，model层与presenter层，和presenter层与view层，均是通过接口进行交互。其中Presenter中同时持有View层的Interface的引用以及Model层的引用，而View层持有Presenter层引用。

![img](http://upload-images.jianshu.io/upload_images/3985563-03352e00ce8b4083.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



# 事件分发机制

### 事件

#### 触摸事件

- MotionEvent.ACTION_DOWN：按下View（所有事件的开始）
- MotionEvent.ACTION_MOVE：滑动View
- MotionEvent.ACTION_CANCEL：非人为原因结束本次事件
- MotionEvent.ACTION_UP：抬起View（与DOWN对应）

### 事件分发过程

**传递顺序是：Activity（Window） -> ViewGroup -> View**

![img](http://upload-images.jianshu.io/upload_images/944365-aa8416fc6d2e5ecd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- super：调用父类方法。
- true：消费事件，即事件不继续往下传递。
- false：不消费事件，事件也不继续往下传递 / 交由给父控件onTouchEvent（）处理。

### 事件分发的协作方法

![img](http://upload-images.jianshu.io/upload_images/944365-a5eeeae6ee27682a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由dispatchTouchEvent() 、onInterceptTouchEvent()和onTouchEvent()三个方法协助完成。

#### dispatchTouchEvent()

**默认情况**

| 对象      | 返回方法                   |
| --------- | -------------------------- |
| Activity  | super.dispatchTouchEvent() |
| ViewGroup | onIntercepTouchEvent()     |
| View      | onTouchEvent()             |

**返回值**

1. 返回true:消费该事件，事件不会在往下传递，后续的事件(MOVE、UP)。
2. 返回false:不消费事件，事件也不会往下传递，而是回传给父控件的onTouchEvent(），对于Activity来说，false=消费事件。但后续事件(MOVE,UP)会继续分发到View

#### onTouchEvent()

**返回值**

1. 返回true：表示自己处理消费这个事件，事件停止传递，后续事件让其处理。
2. 返回false：不处理该事件，事件会往上传给父控件的onTouchEvent()处理，当前view不再接收此后续事件。

#### onIntercceptTouchEvent()

**只有ViewGroup有此方法**

**返回值：**

![img](http://upload-images.jianshu.io/upload_images/944365-6b34c1017b14104d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 三者关系

```java
// 点击事件产生后，会直接调用dispatchTouchEvent（）方法
public boolean dispatchTouchEvent(MotionEvent ev) {

    //代表是否消耗事件
    boolean consume = false;


    if (onInterceptTouchEvent(ev)) {
    //如果onInterceptTouchEvent()返回true则代表当前View拦截了点击事件
    //则该点击事件则会交给当前View进行处理
    //即调用onTouchEvent (）方法去处理点击事件
      consume = onTouchEvent (ev) ;

    } else {
      //如果onInterceptTouchEvent()返回false则代表当前View不拦截点击事件
      //则该点击事件则会继续传递给它的子元素
      //子元素的dispatchTouchEvent（）就会被调用，重复上述过程
      //直到点击事件被最终处理为止
      consume = child.dispatchTouchEvent (ev) ;
    }

    return consume;
   }
//这是viewgroup的
```

#### 滑动冲突的解决

1.外部拦截：在父容器的onInterceptTouchEvent方法内部判断是否拦截MotionEvent.ACTION_MOVE,拦截的判断方法可以使用水平方向与竖直方向的距离差作为判断条件。

2.内部拦截：通过父容器的requestDisallowInterceptTouchEvent方法进行实现。requestDisallowInterceptTouchEvent用于设置是否屏蔽onInterceptTouch方法。

> **父容器**:修改onInterceptTouchEvent，对于MotionEvent.ACTION_DOWN，直接返回false不拦截事件；而MotionEvent.ACTION_MOVE与MotionEvent.ACTION_UP，则返回true拦截事件。
>
> **子View**：修改dispatchTouchEvent方法，1.对于MotionEvent.ACTION_DOWN，设置requestDisallowInterceptTouchEvent为true，即在接下来的事件里屏蔽父容器的onInterceptTouch方法，MOVE与UP会继续分发给该View。2.在MotionEvent.ACTION_MOVE里，判断是否拦截消费，若不消费，再设置requestDisallowInterceptTouchEvent为false即可。



# View的绘制流程

### Measure流程

- 子View：调用measure()方法，内部调用onMeasure()方法，而onMeasure()中调用setMeasuredDimension()设定View的宽高信息，完成View的测量操作。
- ViewGroup(空实现，基础实现思想)：调用measure()方法，内部需要遍历子元素调用其measure方法；然后setMeasuredDimension()设定View的宽高信息。

#### MeasureSpec

代表一个32位int值，高2位代表SpecMode，低30位代表SpecSize。

SpecMode有三类：

- **UNSPECIFIED**:不对View进行任何限制，一般用于系统内部。
- **EXACTLY**:父容器检查到View所需要的精确大小，其最终大小就是SpecSize值。用于LayoutParams的match_parent和具体数值。
- **AT_MOST**：View的大小不能大于SpecSize，对应LayoutParam的wrap_parent。

#### MeasureSpec与LayoutParams父容器的对应关系

- 顶级View（DecorView），其MeasureSpec由窗口的尺寸和其自身的LayoutParams来共同决定。
- 普通View：其MeasureSpec有父容器的MeasureSpec和自身的LayoutParams共同决定。

![img](http://upload-images.jianshu.io/upload_images/3985563-e3f20c6662effb7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

一旦其MeasureSpec确定，onMeasure就可以确定View的测量宽/高了。

### Layout流程

在layout()中使用setFrame()/setOpticalFrame()确定View自身的位置信息，即left，top，right，bottom。再调用onLayout()去调用子View的layout()方法，让子View布局。

- 子View：不需要实现onLayout()，因为没有子布局。
- ViewGroup：需要实现onLayout()，要遍历子View，计算来使用setChildFrame()指定每个子View的位置信息。

#### View的位置参数属性

- left，top：相对于父容器，View左上角的坐标；right，bottom：相对于父容器，View右下角的坐标。表示的是View原始的坐标信息。
- x, y:也是相对容器，View左上角的坐标，而translationX，translationY是View左上角相对于父容器的偏离量。

$$
x = left + translationX
$$

$$
y = top + translationY
$$

### Draw流程

- 绘制背景 background.draw(canvas)
- 绘制自己(onDraw)
- 绘制Children(dispatchDraw)
- 绘制装饰(onDrawScrollBars)

**无论是ViewGroup还是单一的View，都需要实现这套流程，不同的是，在ViewGroup中，实现了 dispatchDraw()方法，而在单一子View中不需要实现该方法。自定义View一般要重写onDraw()方法，在其中绘制不同的样式。**

**ViewGroup默认启用WillNotDraw优化标记位，不绘制任何内容；可通过setWillNoDraw(boolean)来设置改变。**

### 自定义View

- **自定义View**

> 在onDraw()中,实现支持padding
>
> 在onMeasure()中，实现支持wrap_parent

- **自定义ViewGroup**

> 在onMeasure()中，注意处理自己的padding与子View的margin
>
> 在onLayout中，计算出子View的四个坐标值。

**注意**：

自定义View有线程或动画时，需要及时停止。可在View的onDetachedFromWindow方法完成逻辑。

**View的生命周期**：onFinishInflate()->onAttachToWindow->onMeasure->onSizeChanged->onLayout->onDraw->onDetachedFromWindow

# Android性能优化

### 优化方法

1. 布局优化
2. 绘制优化
3. 内存泄漏优化
4. 响应速度优化
5. ListView/RecyclerView及Bitmap优化
6. 线程优化
7. 其他性能优化

### 布局优化*

##### 思想：尽量减少布局文件的层级。

> 布局层级少了，那么Android绘制时的工作量就少了，那么程序的性能自然提高了。

##### 方法：

1. 删除布局中无用的控件和层次，其次由选择地使用性能比较低的ViewGroup。

> 例如：相对于LinearLayout与FrameLayout，RelativceLayout的布局需要花费更多的CPU时间。因此，如果都可以选择，LinearLayout与FrameLayout；如果嵌套情况过多，RelativeLayout会更好。

2. 采用标签ViewStub

> 提供按需加载的功能，当需要时才将ViewStub中的布局加载到内存，**提高程序初始化效率**

3. 避免过度绘制
4. <include/>标签可以封装重复性的布局，减少工作量

```xml
<include android:id="@+id/news_title"
         //可以覆盖导入的布局文件root布局的id
 android:layout_width="match_parent"      android:layout_height="match_parent"
         //嵌入布局的资源引用
 layout="@layout/title"/>
```

5. <merge/>标签可以有效减少View树的层次优化。需要和<include/>

### 绘制优化

##### 思想：View的onDraw方法要避免大量的操作。因为onDraw会被频繁调用。

1. onDraw中不要创建新的局部对象。
2. onDraw中不要做耗时的任务。

### 内存泄露优化*

##### 1.在开发过程中避免写出有内存泄漏的代码

> java有回收机制，会定时回收没有与任何引用关联的对象
>
> 分辨无引用法：1.引用计数法 2.可达性分析法

##### 内存泄漏的类型：

1. 集合类泄漏
2. 单例/静态变量造成的内存泄漏
3. 匿名内部类/非静态内部类：如Handler，Async Task
4. 资源未关闭造成的泄漏

##### 2.通过一些分析工具找出潜在的内存泄漏，然后解决

> 通过MAT(Memory Analyzer Tool)，或者 LeakCanary来检测Android中的内存泄漏。

### 响应速度优化*

##### 思想：避免在主线程中做耗时操作

**Android规定，Activity如果5秒钟之内无法响应屏幕触摸事件或者键盘输入事件就会出现ANR，而BroadcastReceiver如果10秒钟之内还未执行完操作也会出现ANR，而前台Service如果20秒之内未完成创建也会ANR**

### ListView/RecyclerView及Bitmap优化

##### ListView/RecyclerView优化

1. 使用ViewHolder模式来提高效率

```java
//使用ViewHolder
    private static class ViewHolder {
        private ImageView ivIcon;
    }
@Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder viewHolder;
        //判断是否有缓存
        if (convertView == null) {
            //通过LayoutInflate实例化布局
            viewHolder = new ViewHolder();
            convertView = mInflater.inflate(R.layout.item_layout, parent, false);
            viewHolder.ivIcon = (ImageView) convertView.findViewById(R.id.iv_icon);
            convertView.setTag(viewHolder);
        } else {
           //通过tag找到缓存的布局
            viewHolder = (ViewHolder) convertView.getTag();
        }        
        ....
        //加载数据
             viewHolder.ivIcon.setTag(urlString); // 将ImageView与url绑定，用于防止异步加载出现的错位
        return convertView;
    }

```

2. 异步加载：耗时的操作放在异步线程中

使用Glide，Handler，AsyncTask都可以实现。

异步加载产生的错位现象的解决：

- 使用findViewWithTag：

```java
public View getView(int position, View convertView, ViewGroup parent) {
        ....
        //加载数据
       //将url设为imagView的标识位
String urlString = newsBean.newsIconUrl; 
viewHolder.ivIcon.setTag(urlString); // 将ImageView与url绑定     
    }
if (mImageView.getTag().equals(mUrl)) { 
//当url标记和原先设置的一样时，才设置ImageView (对比url)   
mImageView.setImageBitmap((Bitmap) msg.obj);
}
```

- 为图片设置缓存：

使用LrcCache进行图片缓存：

```java
//下面是建立缓存
        int maxMemory = (int) Runtime.getRuntime().maxMemory();  //运行时最大内存
        int cacheSize = maxMemory/4;  //缓存的大小
        mCaches = new LruCache<String, Bitmap>(cacheSize){
            @Override
            protected int sizeOf(String key, Bitmap value) {
                return value.getByteCount();
            }
        };

//将bitmap添加到缓存
    public void addBitmapToCache(String url,Bitmap bitmap){
        if (getBitmapFormCache(url) == null){
            mCaches.put(url, bitmap);
        }
    }

    //从缓存中获取数据
    public Bitmap getBitmapFormCache(String url){
        return mCaches.get(url);
    }
```

3. ListView和RecyclerView的滑动时停止加载和分页加载

```java
@Override
    public void onScrollStateChanged(AbsListView view, int scrollState) {
        switch (scrollState){
            case SCROLL_STATE_IDLE:  //滑动停止时。
                mImageLoader.loadImages(mStart, mEnd);
                break;
            case SCROLL_STATE_TOUCH_SCROLL: //正在滑动时
                mImageLoader.cancelAllTasks();
                break;
            case SCROLL_STATE_FLING: //手指抛动时，即手指用力滑动在离开后ListView由于惯性而继续滑动

                break;
        }
    }

    @Override
    public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) {
        mStart = firstVisibleItem;
        mEnd = firstVisibleItem + visibleItemCount;
        //第一次的时候预加载
        if (mFirstIn && visibleItemCount > 0){
            mImageLoader.loadImages(mStart, mEnd);
            mFirstIn = false;
        }
    }

public void loadImages(int start, int end){
        for (int i = start; i < end; i++){
            String url = NewsAdapter.URLS[i];
            //由缓存中得到bitmap
            Bitmap bitmap = getBitmapFormCache(url);
            if (bitmap == null){
                //当bitmap为空时，由AsyncTask进行加载，并在onPostExecute()方法中setImageBitmap
                NewsAsyncTask task = new NewsAsyncTask(url);
                task.execute(url);
                mAsyncTask.add(task);
            } else {
                //当bitmap不为空时，直接进行setImageBitmap
                ImageView imageView = (ImageView) mListView.findViewWithTag(url);
                imageView.setImageBitmap(bitmap);
            }
        }
    }
```



##### Bitmap优化

主要是对加载图片进行压缩，避免OOM出现

使用Bitmap压缩策略

### 线程优化

采用线程池，避免存在大量的Thread以及减少线程的创建和销毁带来的性能开销。

### 其他性能优化*

1. 不要过度使用枚举，枚举占用的内存空间要比整形大
2. 常量使用static final来修饰
3. 使用一些Android特有的数据结构，比如SparseArray和Pair等
4. 适当采用软引用和弱引用
5. 采用内存缓存和磁盘缓存
6. 采用静态内部类，避免潜在的由于内部类引起的内存泄露

# 进程优先级

### 类型

#### 前台进程

- 处于正在与用户交互的Activity
- 与前台activity绑定的Service
- 前台Service
- 正在执行onCreate，onStart，onDestroy方法的 Service。
- 进程中包含正在执行onReceive（）方法的BroadcastReceiver。

系统中的前台进程并不会很多，而且一般前台进程都不会因为内存不足被杀死。

#### 可视进程

- 为处于前台，但仍然可见的Activity（例如：调用了onPause而还没调用onStop的Activity）。典型情况是：运行activity时，弹出对话框（dialog等），此时的activity虽然不是前台activity，但是仍然可见。
- 可见Activity绑定的Service。（处于上诉情况下的activity所绑定的service）

可视进程一般也不会被系统杀死，除非为了保证前台进程的运行不得已而为之。

#### 服务进程

正在运行已使用 startService() 方法启动的服务且不属于上述两个更高类别进程的进程。

#### 后台进程

- 不可见的activity（调用onstop（）之后的activity）

后台进程不会影响用户的体验，为了保证前台进程，可视进程，服务进程的运行，系统随时有可能杀死一个后台进程。

#### 空进程

- 任何没有活动的进程

系统会杀死空进程，但这不会造成影响。空进程的存在无非为了一些缓存，以便于下次可以更快的启动。

### OOM_ADJ

Android系统依据自身的一套进程回收机制，杀死一些进程，以腾出内存。称为 Low Memory Killer，它是根据 OOM_ADJ 阈值级别触发相应力度的内存回收的机制。

![img](https://upload-images.jianshu.io/upload_images/4943911-9b1bf14b2e2d1ffc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/662/format/webp)

- 进程的oom_adj越大，表示此进程优先级越低，越容易被杀回收；
- 普通app进程的oom_adj>=0,系统进程的oom_adj才可能<0
- 白色部分一般是ROM自带的app或者服务
- 红色部分代表比较容易被杀死的 Android 进程（OOM_ADJ>=4）
- 绿色和红色是android进程

### 进程保活

1. app进程长时间停留在后台，被kill

> app运行一个前台Service，可使OOM_ADJ数值由4降低到1
>
> 缺陷：4.1后Service.startForeground() 会强制弹出通知栏，解决办法是再启动一个service和推送共用一个通知栏，然后stop这个service使得通知栏消失。而7.1后已修复这个bug，目前没有解决办法

2. 进入锁屏状态后，被省电机制会kill后台进程

> 注册广播监听锁屏和解锁事件， 锁屏后启动一个1像素的透明Activity，这样直接把进程的oom_adj数值降低到0，0是android进程的最高优先级。 解锁后销毁这个透明Activity。
>
> 缺陷：可能导致卡顿

### 进程拉活

1. 通过app广播拉活

> 当几个App都集成了同一家的推送，只要有一个App起来，就会发送一个广播，这样其它的App接收到这个广播之后，开启一个服务，就把进程给启动起来了

2. 通过系统广播拉活

> 缺陷:
>
> 1. 广播接收器被管理软件、系统软件通过“自启管理”等功能禁用的场景无法接收到广播，从而无法自启。
> 2. 系统广播事件不可控，只能保证发生事件时拉活进程，但无法保证进程挂掉后立即拉活。

3. 将Service设置为 START_STICKY，利用系统机制在 Service 挂掉后自动拉活。

> 缺陷：一旦在短时间内 Service 被杀死4-5次，则系统不再拉起

4. 使用JobSchedule拉活

> JobScheduler允许在特定状态与特定时间间隔周期执行任务。可以利用它的这个特点完成保活的功能，效果类似开启一个定时器，与普通定时器不同的是其调度由系统完成。
>
> 缺陷：在Android5.0之后推出的，在5.0之前无法使用

5. 双进程守护

> 将Service以bind的方式开启，在被杀死时，在回调的ServiceConnection的onServiceDisconnected中实现相互拉活。
>
> 可以在主进程中进行有一个LocalService，在子进程中有RemoteService。LocalService中以bind和start方式启动RemoteService，同时RemoteService以bind和start方式启动LocalService。并且在它们各自的ServiceConnection的onServiceDisconnected方法中重新bind和start。

6. 推送拉活

> 接入推送，实现进程被推送唤醒。

7. 利用Native进程拉活

> 利用 Linux 中的 fork 机制创建 Native 进程，在 Native 进程中监控主进程的存活，当主进程挂掉后，在 Native 进程中立即对主进程进行拉活。
>
> 缺陷:主要适用于 Android5.0 以下版本手机

#### 总结

最好的解决方式是加入白名单最可靠

# Android的缓存策略

### LRU缓存算法

最近最少使用的算法：核心思想是当缓存满时，会优先淘汰那些最近最少使用的缓存对象。

采用LRU算法的缓存有两种：LrhCache和DisLruCache，分别用于实现内存缓存和硬盘缓存

### LruCache介绍

LruCache是个泛型类，主要算法原理是把对象用强引用存储在 LinkedHashMap中。当缓存满时，把最近最少使用的对象从内存中移除，并提供了get和put方法来完成缓存的获取和添加操作。

(LinkedHashMap是由数组+双向链表的数据结构来实现的,用链表实现保持插入的顺序)

```java
public LinkedHashMap(int initialCapacity,
    float loadFactor,
    boolean accessOrder)
    //其中accessOrder设置为true则为访问顺序，为false，则为插入顺序。
    //accessOrder设置为true则正是满足LRU算法
```

### LruCache使用

```java
int maxMemory = (int) (Runtime.getRuntime().totalMemory() / 1024);
int cacheSize = maxMemory / 8;
mMemoryCache = new LruCache<String, Bitmap>(cacheSize) {
    @Override
    protected int sizeOf(String key, Bitmap value) {
        return value.getRowBytes() * value.getHeight() / 1024;
    }
};
```

1. 设置LruCache缓存的大小，一般为当前进程可用容量的1/8。 
2. 重写sizeOf方法，计算出要缓存的每张图片的大小。

### LruCache原理

**LruCache中维护了一个集合LinkedHashMap，该LinkedHashMap是以访问顺序排序的。当调用put()方法时，就会在结合中添加元素，并调用trimToSize()判断缓存是否已满，如果满了就用LinkedHashMap的迭代器删除队尾元素，即近期最少访问的元素。当调用get()方法访问缓存对象时，就会调用LinkedHashMap的get()方法获得对应集合元素，同时会更新该元素到队头。**

### DisLruCache

DisLruCache不同于LruCache，LruCache是将数据缓存到内存中去，而DiskLruCache是外部缓存。

DiskLruCache不是google官方所写，但是得到了官方推荐，DiskLruCache没有编写到SDK中去。

[https://github.com/JakeWharton/DiskLruCache](https://link.jianshu.com/?t=https://github.com/JakeWharton/DiskLruCache)

# Android热修复原理

