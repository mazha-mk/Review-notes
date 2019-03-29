# 并发

### 区别

**并发**：同一时间段有几个程序都处于已经启动到运行完毕之间，并且这几个程序都在同一个处理机上运行，**并发的两种关系是同步和互斥**。
**互斥**：进程之间访问临界资源时相互**排斥**的现象。
**同步**：进程之间存在**依赖关系**，一个进程结束的输出作为另一个进程的输入。
**并行**：同一时刻同时执行两个或多个处理的一种计算方法。
**异步**：和同步相对，同步是顺序执行，而异步是彼此独立，在等待某个事件的过程中继续做自己的事，不要等待这一事件完成后再工作。
**异步和多线程**：不是同等关系，异步是目的，多线程只是实现异步的一个手段，实现异步可以采用多线程技术或者交给其他进程来处理。

### 创建线程的三种方式

1. 继承Thread类创建线程类

~~~java
import java.lang.Thread;
class Threads extends Thread{
    @Override
    public void run(){
        //balabala
    }
}
~~~

2. 通过Runnable接口创建线程类

~~~java
import java.lang.Thread;
import java.lang.Runnable;
new Thread(new Runnable(){
            @Override
            public void run(){
				//balabala
            }
        }).start();
~~~

3. 通过实现Callable的FutureTask创建线程

~~~java
import java.util.concurrent.Callable;
import java.util.concurrent.Future;
import java.util.concurrent.FutureTask;
FutureTask<Integer> ft = new FutureTask<>(new Callable<Integer>(){
            @Override
            public Integer call(){
               //balabala
               return null;
            }
        });
        new Thread(ft).start();
        System.out.println(ft.get());//会阻塞直到获得返回
~~~

#### 区别

1. Runnable、Callable接口：好处是只是实现了接口，还可以继承其他类；访问当前线程，要使用Thread.currentThread()方法。
2. 继承Thread类：直接使用this即可获得当前线程；继承了Thread类，所以不能再继承其他父类。

### ==线程池==

#### 优势

1. 通过重用已存在的线程，降低系统线程创建和销毁造成的资源消耗
2. 无需创建线程直接使用，提高系统响应速度
3. 方便线程并发数的管控。

#### ExecutorService

线程池的基接口。

ExecutorService service = new ThreadPoolExecutor(....);

#### ThreadPoolExecutor

参数含义:

1. corePoolSize:线程池中的核心线程数,默认情况会一直存活在线程池中。

> 如果ThreadPoolExecutor设置allowCoreThreadTimeOut属性设为true，超时时间就由keepAliveTime来指定，超时则被终止。

2. maximumPoolSize：线程池中所容纳的最大线程数，即核心线程数+非核心线程数的容量。
3. keepAliveTime：只有设置了allowCoreThreadTimeOut属性设为true时，才有效。
4. unit：一个时间单位的枚举。
5. workQueue：线程池中用于保存等待执行的任务的阻塞队列，有很多种。

| **阻塞队列**          | **说明**                                                     |
| --------------------- | ------------------------------------------------------------ |
| ArrayBlockingQueue    | 基于数组实现的有界的阻塞队列,该队列按照FIFO（先进先出）原则对队列中的元素进行排序。 |
| LinkedBlockingQueue   | 基于链表实现的阻塞队列，该队列按照FIFO（先进先出）原则对队列中的元素进行排序。 |
| SynchronousQueue      | 内部没有任何容量的阻塞队列。                                 |
| PriorityBlockingQueue | 具有优先级的无限阻塞队列。                                   |



6. threadFactory：为线程池提供新线程的创建；是一个接口，里面只有一个newThread方法。 默认为DefaultThreadFactory类。
7. handler:实现RejectedExecutionHandler接口的对象;

> 当任务队列已满并且线程池中的活动线程已经达到所限定的最大值或者是无法成功执行任务，这时候ThreadPoolExecutor会调用RejectedExecutionHandler中的rejectedExecution方法。

下面是在ThreadPoolExecutor中提供的四个可选值。

| **可选值**          | **说明**                                   |
| ------------------- | ------------------------------------------ |
| CallerRunsPolicy    | 只用调用者所在线程来运行任务。             |
| AbortPolicy         | 直接抛出RejectedExecutionException异常。   |
| DiscardPolicy       | 丢弃掉该任务，不进行处理。                 |
| DiscardOldestPolicy | 丢弃队列里最近的一个任务，并执行当前任务。 |

##### execute和submit

两种方式都是向线程池提交一个任务。

execute():没有返回值；

submit():会返回一个future对象，可通过future来判断任务是否执行成功，获取返回值。

##### shutdown()和shutdownNow()

两种方式都是关闭线程池。

shutdown()：将线程池状态设置成SHUTDOWN状态，然后中断所有没有正在执行任务的线程。

shutdownNow():状态设置成STOP状态，然后中断所有任务(包括正在执行的)的线程,，并返回等待执行任务的列表。

##### ==ThreadPoolExecutor执行流程==

提交任务->核心线程池已满？->工作队列已满？->线程池已满？->按饱和策略处理。

![img](http://upload-images.jianshu.io/upload_images/3985563-c05771982ad27f86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### ==Executors类==

提供了四种具有不同功能常见的线程池的静态创建方法

##### 1.newFixedThreadPool:

**所容纳最大的线程数就是我们设置的核心线程数**它能够更快速的响应外界请求，不存在超时机制。

##### 2.newCachedThreadPool：

**核心线程数为0，最大线程数可以任意大**当线程池中的线程都处于活动状态的时候，线程池就会创建一个新的线程来处理任务。该线程池中的线程超时时长为60秒，所以当线程处于闲置状态超过60秒的时候便会被回收。

##### 3.newScheduledThreadPool：

**核心线程数固定，最大线程数可以任意大，schedule()方法进行延迟任务**有强大的定时执行任务功能。

##### 4.newSingleThreadExecutor：

**只有一个核心线程,线程最大数为1，任务队列大小没有限制**任务执行之间我们不需要处理线程同步的问题。

#### 技巧

任务类别：

1. CPU密集型任务

> 线程池中线程个数应尽量少，如配置N+1个线程的线程池。

2. IO密集型任务

> 由于IO操作速度远低于CPU速度，为提高cpu利用率v,可配置如2*N。

3. 混合型任务。(N代表CPU个数)

> 两类任务执行时间相差无几，拆分；两类任务执行时间有数据级的差距，没必要拆分。



### 死锁条件

1. 互斥条件：一个资源每次只能被一个线程使用。
2. 请求与保持条件：一个线程因请求资源而阻塞时，对已获得的资源保持不放。
3. 不剥夺条件：线程已获得的资源，在未使用完之前，不能强行剥夺。
4. 循环等待条件：若干线程之间形成一种头尾相接的循环等待资源关系。

### ==两种锁：synchronized和ReentrantLock==

#### 可重入锁

synchronized和ReentrantLock都是重入锁:调用method1()方法时，已经获得了锁，此时内部调用method2()方法时,由于本身已经具有该锁，所以可以再次获取。（即可以被单个线程多次获取。）

#### 公平锁与非公平锁

**公平的定义**：保证线程按照时间的先后顺序执行。

synchronized锁是非公平锁。(随机性)

ReentrantLock可以实现公平锁。()，在构造方法传入true，为公平锁；传入false，为非公平锁。

#### synchronized关键字

##### 特性

表现形式：代码块同步和方法同步。

本质上都是获取一个对象的监视器（monitor）的所有权，任意一个对象都拥有自己的监视器。

##### 使用场景

1. public synchronized void method1：锁的是该对象
2. synchronized(xxx.class){  }:锁的是该类.
3. public synchronized static void method3:锁住的是该类,不同对象调用该方法会产生互斥。
4. synchronized(o) {}：锁是o对象，可以是一个任何Object对象或数组。

##### 同步方法和同步块

同步方法默认使用this或者当前类做为锁。

同步代码块可以选择以什么来加锁，比同步方法更精确，我们可以选择只有会在同步发生同步问题的代码加锁，而并不是整个方法。

#### ReentrantLock锁：

Lock接口,JAVA SE5.0之后并发包中新增了Lock接口用来实现锁的功能,在使用时需要显式地获取和释放锁.

#### ==synchronized与ReentrantLock的区别==

==1==.ReentrantLock是Lock接口的实现类，而synchronized是Java中的关键字。

==2==.synchronized在发生异常时，会自动释放线程占用的锁，不会造成死锁。而Lock没有主动释放机制，需要在finally中unlock()，否则会产生死锁。

==3==.lock可以让等待的线程中断等待，而synchronized，等待的线程会一直等待。（ReentrantLock是可中断锁，synchronized是不可中断锁）

4.Lock可以知道有没有成功获取锁，而synchronized却无法办到。

==5==.synchronized是非公平锁，而ReentrantLock可以是公平锁。



### ==线程间通信的两种方式==

#### Object类的wait()/notify()

1. wait():让当前的线程进入等待，释放锁。

> 1. 持有锁才能调用wait(),否则会抛出异常。
> 2. 即wait()方法的调用必须放在synchronized方法或synchronized块中。

2. wait(long):让当前的线程进入等待，释放锁，不过等待时间为long，超过这个时间没有对当前线程进行唤醒，将自动唤醒。
3. notify()：当前线程执行完毕后释放锁，并从其他线程中唤醒其中一个继续执行。

> 1. 使用时，唤醒的线程的选择时随意的。
> 2. 被唤醒的线程并不能执行的，需要等到当前的线程释放锁了，才能执行。

4. notifyAll():当前线程执行完毕后释放锁，将唤醒所有等待状态的线程。

==**wait与sleep的区别**==：wait()与sleep():wait会释放锁，而sleep并没有释放锁。

#### Condition实现等待/通知：

1. condition.await()—>lock.wait()
2. condition.await(long time, TimeUnit unit)—>lock.wait(long timeout)
3. condition.signal()—>lock.notify()
4. condition.signaAll()—>lock.notifyAll()

**特殊之处**：synchronized相当于整个ReentrantLock对象只有一个单一的Condition对象情况。而一个ReentrantLock却可以拥有多个Condition对象，来实现通知部分线程。

> Synchronized中，所有的线程都在同一个object的条件队列上等待。而ReentrantLock中，每个condition都维护了一个条件队列。

> Condition是与Lock绑定的，如果是公平锁，线程为按照FIFO的顺序从Condition.await中释放，如果是非公平锁，那么后续的锁竞争就不保证FIFO顺序了。 

相比一生产一消费的模式，改动了两处。①signal()-->signalAll()避免进入“假死”状态。②if()判断-->while()循环，重新判断条件，避免逻辑混乱。

### ==并发程序正确性==

**要想并发程序正确地执行，必须要保证原子性、可见性以及有序性。只要有一个没有被保证，就有可能会导致程序运行不正确。**

#### JAVA内存模型

JVM控制的主内存(共享变量)，而每个JAVA线程都有一个自己的工作内存(共享变量的副本)，不同线程之间无法直接访问对方的工作内存，并且，线程对变量的所有操作（读取，赋值）都必须在工作内存中进行。

![img](http://upload-images.jianshu.io/upload_images/3985563-8a3d9a1b94b97e83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 原子性

即一个操作或者多个操作，要么全部执行，要么都不执行，不能被打断。

> 1. 对基本数据类型的变量的*读取*和*赋值*操作是原子性操作。
> 2. 只有简单的读取、赋值（而且必须是将数字赋值给某个变量，变量之间的相互赋值不是原子操作）才是原子操作。

##### JAVA实现原子性

要实现更大范围操作的原子性，可以通过**互斥锁**来实现，因为保证了只有一个线程执行代码块，就不存在原子性的问题了。

#### 可见性

指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。

##### JAVA实现可见性：

使用volatile关键字。

#### 有序性

程序执行的顺序按照代码的先后顺序执行。

##### JAVA不能保证有序性：

JVM会对输入代码进行优化，可能会发生**指令重排序**，但指令重排序会考虑指令之间的数据依赖性，因此不会对结果造成影响。所以指令重排序不会影响单个线程的执行，但是会影响到线程并发执行的正确性。

##### JAVA实现可见性：

volatile关键字可以保证一定的有序性。
synchronized和Lock可以保证有序性。

#### ==先行发生原则(happens-before原则)==

Java内存模型具备一些先天的“有序性”，即happens-before原则（先行发生原则）：

1. 程序次序规则：一个线程内，按照代码顺序，书写在前面的操作先行发生于书写在后面的操作
2. 锁定规则：一个unLock操作先行发生于后面对同一个锁的lock操作
3. **volatile变量规则**：对一个变量的写操作先行发生于后面对这个变量的读操作
4. 传递规则：如果操作A先行发生于操作B，而操作B又先行发生于操作C，则可以得出操作A先行发生于操作C
5. 线程启动规则：Thread对象的start()方法先行发生于此线程的每个一个动作
6. 线程中断规则：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生
7. 线程终结规则：线程中所有的操作都先行发生于线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行
8. 对象终结规则：一个对象的初始化完成先行发生于他的finalize()方法的开始。

**其中1因为JVM会对代码进行指令重排序，所以在多线程中无法保证有序性。**

#### volatile关键字：保证可见性。

1. 保证了不同线程对这个变量进行操作时的可见性。
2. 禁止进行指令重排序（其实是保证了volatile变量的前面的代码操作会在它前面执行，后面的在它后面执行；至于它前面的代码还是有可能会有指令排序，后面的也一样）。
3. 不保证原子性。

#### ==实现并发正确性==

##### JAVA的Atomic*类

保证了基础类型对象的一些操作的原子性

例如AtomicInteger对自增，自减，以及加法操作，减法操作进行了封装，保证原子性。atomic是利用**CAS来实现原子性操作的**。

##### AtomicReference类

Java1.5开始JDK提供了来**保证引用对象之间的原子性**，你可以把多个变量放在一个对象里来进行CAS操作。

##### ==悲观锁==

在某个资源不可用的时候，就将cpu让出，把当前等待线程切换为阻塞状态。等到资源(比如一个共享数据）可用了，那么就将线程唤醒，让他进入runnable状态等待cpu调度。

##### ==乐观锁==

每次不加锁而是假设修改数据之前其他线程一定不会修改，如果因为修改过产生冲突就失败就重试，直到成功为止。 （当数据争用不严重时，乐观锁效果更好）CAS操作就是乐观锁思想。

##### ==CAS 操作==

**包含三个操作数——内存位置（V）、预期原值（A）和新值(B)**。执行CAS操作的时候，将V的值与A比较，如果相匹配，那么处理器会自动将该位置值更新为新值。否则，处理器不做任何操作。

##### CAS存在的问题：

1. ABA问题：不能保证内存位置的A值就是预期原值的A值，可能中间存在了B操作。

   > JDK1.5的AtomicStampedReference可以解决ABA问题，通过为每次更新追加上版本号的方式。

2. 自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。

3. 只能保证一个共享变量的原子操作。

------

### BlockingQueue(线程安全)

是一个接口：不接受 null 元素；可以是限定容量的；支持Collection 接口；内部排队方法**使用内部锁，因此是是线程安全的**。

|             | Throws exception | Returns special value | Blocks | Times out            |
| ----------- | ---------------- | --------------------- | ------ | -------------------- |
| **Insert**  | add(e)           | offer(e)              | put(e) | offer(e, time, unit) |
| **Remove**  | remove()         | poll()                | take() | poll(time, unit)     |
| **Examine** | element()        | peek()                | 无     | 无                   |

#### 实现BlockingQueue接口的类

##### 1.ArrayBlockingQueue

大小固定的Blocking Queue,由数租维护，循环队列。支持公平锁，**底层有由一个ReentrantLock和两个Condition实现**，默认是非公平锁。

##### 2.LinkedBlockingQueue

Executors.newFixedThreadPool()底层所使用的阻塞队列。与ArrayBlockingQueue不同，**底层由两个ReentrantLock实现，并用AtomicInteger类型的队列中的元素数量变量保证了修改时的原子性**，因此，实现了同一时刻既消费、又生产，能够做到并行处理。

##### 3.DelayQueue

没有大小限制的队列，元素需要实现 java.util.concurrent.Delayed接口；元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素。因此，（生产者）永远不会被阻塞。

##### 4.PriorityBlockingQueue

没有边界的队列，所以不会阻塞生产者；元素必须实现 java.lang.Comparable 接口。

##### 5.SynchronousQueue

只能够容纳单个元素。因此，当没有元素时，消费者阻塞；有元素时，生产者阻塞。

### ==HashTable==

基本可以等价于HashMap。

不过相比于HashMap,HashTable使用了synchornized关键字实现了线程安全性。

Hashtable和HashMap有几个主要的不同：线程安全以及速度。

1. sychronized意味着在一次仅有一个线程能够更改Hashtable。
2. 在单线程环境下它比HashMap要慢。
3. HashTable不允许key和value为null

### ==ConcurrentHashMap==

HashMap是非线程安全的。可使用HashTable和Collections.synchronizedMap(HashMap)来解决。两个都是对读写进行加锁操作。性能不好。

#### ConcurrentHashMap JDK1.6 

采用**分段锁的机制**，底层采用数组+链表+红黑树的存储结构。

包含两个核心静态内部类Segment和HashEntry。

1. Segment继承ReentrantLock，是锁，守护每个散列映射表的若干个桶。
2. HashEntry是用来封装映射表的键值对。
3. 每个桶是由若干个 HashEntry 对象链接起来的链表。

![img](http://upload-images.jianshu.io/upload_images/2184951-728c319f48ebe35a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

ConcurrentHashMap定位一个元素的过程需要进行两次Hash操作：

1. Hash定位到Segment
2. Hash定位到元素所在的链表的头部。

这一种结构的带来的副作用是Hash的过程要比普通的HashMap要长

#### ConcurrentHashMap JDK1.8 

利用CAS+Synchronized来保证并发更新的安全，底层依然采用数组+链表+红黑树的存储结构。

![img](http://upload-images.jianshu.io/upload_images/2184951-3d2365ca5996274f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

~~~java
class Node<K,V> implements Map.Entry<K,V>{
    final int hash;
    final K key;
    volatile V val;
    volatile Node<K,V> next;
    //... 省略部分代码}
}
~~~

Java8 ConcurrentHashMap结构基本上和Java8的HashMap一样，不过通过大量的CAS操作，volatile和synchronized保证线程安全性。

 相比于HashTable 和同步包装器包装的 HashMap，对容器的访问变成串行化；ConcurrentHashMap支持给定数量的并发更新。

