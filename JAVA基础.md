# 容器

### 集合与数租

Java集合类：Set、List、Queue和Map四种体系。
Java集合和数组的区别：1.数组定长，集合不定长。2.数组元素可以是基本类型，也可以是对象引用；而集合只可以是对象引用。

### Collection接口

Collection接口是Set,Queue,List的父接口；其中iterator()方法，该方法的返回值是Iterator，迭代器，用来遍历集合中的元素，有 boolean hasNext()和 E next()方法。

![img](http://upload-images.jianshu.io/upload_images/3985563-e7febf364d8d8235.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Set

其中Set代表**无序、不可重复的集合**；
Map里的key是不可重复的，key用户标识集合里的每项数据。
Set集合不允许包含相同的元素，若输入已含有的元素add()会返回false；可使用 Set<K> keySet()获取键的视图。

### List

List是**有序集合**，默认从0开始按添加顺序设索引，在Collection基础上添加了一些根据索引来操作元素的方法。

#### ArrayList

**以数组实现，能自动扩容，默认每次扩容0.5倍大小，可以认为是“动态数组”**。缺点是:如果按下标add(i,e), remove(i), remove(e)，则要用System.arraycopy()来移动部分受影响的元素，性能就变差了，这是基本劣势；其中，自动扩容也是使用的System.arraycopy()来实现。

#### Vector

**以数组实现，能自动扩容，默认每次扩容1倍大小，同时实现了线程安全**

#### LinkedList

**以双向链表实现,按下标访问元素** ，get(i)/set(i,e)要遍历链表将指针移动到位(如果i>数组大小的一半，会从末尾移起)。因此获取节点的复杂度为O(n/2)。

### Queue	

Queue集合“先进先出”(FIFO),新元素插入(offer)到队列的尾部，访问元素(poll)(peek)操作。

### Stack

 **Stack栈先进后出，后进先出；Queue先进先出**

### Map

![img](http://upload-images.jianshu.io/upload_images/3985563-06052107849a7603.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### ==HashMap==

**一个线性的数组实现的**

![img](https://ss1.baidu.com/6ONXsjip0QIZ8tyhnq/it/u=2742330931,2750117589&fm=173&app=25&f=JPEG?w=494&h=274&s=7BA83063F3C349490EDDE1DA000080B1)

**参数：（容量(Capacity)和负载因子(Load factor)）。**

**hash函数：高16bit不变，低16bit和高16bit做了一个异或。**

![img](https://cloud.githubusercontent.com/assets/1736354/6957712/293b52fc-d932-11e4-854d-cb47be67949a.png)

可以接收null的键值，是非同步的，不保证数据有序。可使用Set<Map.Entry<K,V>> entrySet() 获得Map.Entry进行遍历。

**put():**先hashCode()获取code在用hash函数算出index值,无碰撞则存入，若碰撞了，已链表或红黑树存入bucket桶里；

**get():**获取key的index，调用equals()方法确定键值对，命中获取，未命中则若是链表，O(n),若是树，则O(logn)。

（一个bucket桶中碰撞冲突的元素超过某个限制(默认是8)，则使用红黑树来替换链表，从而提高速度。）

**resize：**重新resize一个bucket扩充为2倍的HashMap，之后每个节点重新计算index，再把节点再放到新的bucket中。

#### TreeMap

**使用红黑树实现,好处是有不错的平衡性**

因此速度能达到log(n)的水平，==可以保持key的大小顺序(即树的中序遍历(LDR))==。

##### 红黑树

特性：

1. 节点是红色或黑色

2. 根节点一定是黑色

3. 每个叶节点都是黑色的空节点(NULL节点)

4. 每个红节点的两个子节点都是黑色的

5. 从任一节点到其每个叶子节点的所有路径都包含相同数目的黑色节点

   正是由于这些原因使得红黑树是一个平衡二叉树。

#### LinkedHashMap

**是Hash表和链表的实现**

并且依靠着双向链表,==保证了迭代顺序是插入的顺序==。与父类HashMap不同基本是在put()和get()等操作中加了维护那个具有访问顺序的双向链表。



# 泛型

### 语法糖

主要有泛型、变长参数、条件编译、自动拆装箱、内部类等；
虚拟机并不支持这些语法，它们在编译阶段就被还原回了简单的基础语法结构，这个过程成为解语法糖。

### 定义

Java 泛型：使得在**编译阶段**完成一些**类型转换**的工作，避免在运行时强制类型转换而出现ClassCastException，即类型转换异常。

#### 优点

1. 类型安全。

2. 消除了代码中许多的强制类型转换，增强了代码的可读性。

3. 为较大的优化带来了可能。

### 语法

#### 1.泛型方法

就是在声明方法时定义一个或多个类型形参。
修饰符<T, S> 返回值类型 方法名（形参列表）｛
   方法体
}

#### 2.类型通配符

一个问号（**？**), 它的元素类型可以匹配任何类型。

##### 上限通配符(? extends xxx)

使用extends关键字指定这个类型必须是**继承某个类**，或者实现某个接口，也可以是这个类或接口**本身**。

例：它表示集合中的所有元素都是Shape类型或者其子类
List<? extends Shape>

##### 下限通配符(? super xxx)

使用super关键字指定这个类型必须是是**某个类的父类**，或者是某个接口的父接口，也可以是这个类或接口**本身**。

例：它表示集合中的所有元素都是Circle类型或者其父类
List <? super Circle>

### 其他

#### ==类型擦除==

由于Java泛型只是作用于代码编译阶段,不管为泛型的类型形参传入哪一种类型实参,在编译时，正确检验泛型结果后，会将泛型的相关信息擦出，因而泛型信息不会进入到运行时阶段。

#### 注意

1. 在静态方法、静态初始化块或者静态变量的声明和初始化中不允许使用类型形参。由于系统中并不会真正生成泛型类，所以instanceof运算符后不能使用泛型类。
2. 泛型类派生子类：使用泛型接口、泛型父类派生子类时不能再包含类型形参，需要传入具体的类型，或者不指定具体类型(系统会当成Object)。

# 反射

反射可以帮助我们在**动态运行**的时候，对于任意一个类，可以获得其所有的方法（包括 public protected private 默认状态的），所有的变量 （包括 public protected private 默认状态的）。

### 通过Java反射查看类信息

#### 获得Class对象

1. class1 = Class.forName("com.lvr.reflection.Person");

   >  第一种方式 通过Class类的静态方法——forName()来实现

2. class1 = Person.class; 

   > 第二种方式 通过类的class属性

3. Person person = new Person(); Class<?> class1 = person.getClass(); 

   >  第三种方式 通过对象getClass方法

**Class对象有很多的方法可以获取该Class对象的成员变量、方法、构造函数、注解以及Class对象的信息。**

#### 生成类的实例对象

1. Object obj = class1.newInstance();

   > 第一种方式 Class对象调用newInstance()方法生成 

2. Constructor<?> constructor = class1.getDeclaredConstructor(String.class);obj = constructor.newInstance("hello"); 

   >  第二种方式 对象获得对应的Constructor对象，再通过该Constructor对象的newInstance()方法生成

#### 关于方法的访问权限

调用类的方法时，如果需要调用某个对象的private方法时，可以先调用Method对象的`setAccessible(boolean flag)`，设为true，取消Java语言的访问权限检查。

#### 关于泛型的成员变量 

对于获取成员变量的类型：普通类型的 Field 的数据类型可以通过 Field 的 getType() 方法获取；而泛型参数的类型的 Field的泛型类型，则可以通过Field 的getGenericType() 方法获取。

# 注解(Annotation)

### 元数据

是数据进行说明描述的数据，即程序元素的额外信息。

### Java内建注解

1. @Override ：用于告知编译器，我们需要覆写超类的当前方法。

2. @Deprecated：用于告知编译器，某一程序元素(比如方法，成员变量)不建议使用了（即过时了，标志的变量被使用时会有删除线提示）。

3. @SuppressWarnings：用于告知编译器忽略特定的警告信息；该注解有方法value(）,可支持多个字符串参数，用户指定忽略哪种警告。

4. **@FunctionalInterface**：用户告知编译器，检查这个接口，保证该接口是函数式接口，即只能包含一个抽象方法。

### 用与修饰其他注解的注解

1.@Documented：用户指定被该元Annotation修饰的Annotation类将会被javadoc工具提取成文档。

2.@Inherited：被它修饰的Annotation将具有继承性。

3.@Retention：表示该注解类型的注解保留的时长。

> (SOURCE:仅存在JAVA源文件；CLASS：存在JAVA源文件和CLASS字节码文件，运行时VM不保留；RUNTIME:都存在，可通过反射读取注解)

4.@Target：表示该注解类型的所适用的程序元素类型。

![img](http://upload-images.jianshu.io/upload_images/3985563-7b457df2143fa5dd.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

JAVA8拓宽注解的应用场景：新增了ElementType.TYPE_USER和ElementType.TYPE_PARAMETER；注解几乎可以使用在任何元素上：局部变量、接口类型、超类和接口实现类，甚至可以用在函数的异常定义上。

### 自定义注解

public @interface MyAnnotataion{
    String name();
    String website() default "hello";
    int revision() default 1;
}

ps：注解方法不带参数，注解方法可有默认值。

注解解析：接口AnnotatedElement，内有获取注解的方法。几个实现类：Class，Constructor，Field，Method，Package都有它的实现。

---------------------

# IO

### 编码与路径分隔符

**Java采用unicode编码**，2个字节来表示一个字符；C语言中采用ASCII，一个字符通常占1个字节，而unicode向下兼容ASCII。
utf-8编码方案：按照unicode编码编号，为了节省资源，采用变长编码，编码长度从1个字节到6个字节不等。

**路径分隔符**：windows：'' /'' ''\ ''都可以；linux/unix： "/"；
Java中''\ ''代表转义字符，一般推荐使用代码`File.separator`，表示跨平台分隔符。

可以通过File[] files = file.listFiles(new FilenameFilter()）对目录进行扫描（File类有很多用于扫描的方法）。

### 字节流和字符流

##### 区别：

1. 读写单位不同：字节流以字节（8bit）为单位，字符流以字符为单位，根据码表映射字符，一次可能读多个字节。

2. 处理对象不同：字节流能处理所有类型的数据（如图片、avi等），而字符流只能处理字符类型的数据。

处理流来包装节点流是一种典型的装饰器设计模式

#####  四大基类

InputStream,Reader,OutputStream以及Writer。字符流字节流的类其中的方法不同之处只不过读取的数据单元不同。

> Reader,Writer的单元是char,而InputStream,OutputStream的单元是byte

![img](http://upload-images.jianshu.io/upload_images/3985563-38c3ea4562d6dbe3.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

##### 注意

1. 对于输入流，要调用close()方法关闭，因为程序里打开的IO资源不属于内存资源，垃圾回收机制无法回收该资源。

2. 对于输出流，关闭输出流除了保证资源被回收，还会自动执行输出流的flush()方法。

3. public FileOutputStream(String name, boolean append)中的append指的是是否将流内容写入到file文件的末尾，而不是删除file文件内容重新开头。

### RandomAccessFile

支持读写，支持“随机访问”。重要使用场景就是网络请求中的多线程下载及断点续传。

#### Mode

1. "r"

2. "rw"

3. "rws":要求对文件内容和元数据的每个更新都同步写入储存

4. "rwd"只要求文件内容每个更新都同步写入存储。

# NIO

基于通道(Channel)和缓冲区(Buffer)进行操作，Selector。
*数据总是从通道读取到缓冲区中，或者从缓冲区写入通道也类似*

### NIO具体类

#### Buffer

三个参数：

1. Capacity：buffer固定的大小。

2. Position：光标位置，当为写模式，最大是Capacity-1；当为读模式，最大是limit-1。

3. Limit：写模式时，等于Capacity；读模式时，limit则代表我们所能读取的最大数据量。

![buffers-modes.png](http://upload-images.jianshu.io/upload_images/3985563-74b53331f13ac591.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

##### 子类：

![img](http://upload-images.jianshu.io/upload_images/3985563-0f18367164c56cbd.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

#### Channel

通道可以读也可以写。

##### 子类：

FileChannel,DatagramChannel,SocketChannel,ServerSocketChannel；

> （除FileChannel）可通过设置SelectableChannel configureBlocking(boolean block)：调整此通道的阻塞模式，设置为blocking或者non-blocking。

#### Selector

继承Thread，实现单线程管理多个channels,也就是可以管理多个网络链接。
Channel使用时，必须时non-blocking的。

### blocking和non-blocking的区别

blocking IO会一直block住对应的进程直到操作完成，而non-blocking IO在kernel还准备数据的情况下会立刻返回，但当数据准备好要拷贝到用户内存时，还是会被block。

### synchronous IO和asynchronous IO的区别

synchronous IO做”IO 操作”的时候会将process阻塞。

而事实上，blocking和non-blocking都属于synchronous IO。

JDK1.7才增加asynchronous IO：AsynchronousServerSocketChannel等等。

![img](http://upload-images.jianshu.io/upload_images/3985563-3f7ade558f749a61.gif?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

# 异常

###  关键字

**throw**：用于抛出异常。
**throws**：用在方法签名中，用于声明该方法可能抛出的异常。

**finally**：

1. finally块的语句在try或catch中的return语句执行之后，返回之前执行。

2. finally里的修改语句可能影响也可能不影响try或catch中return已经确定的返回值。

3. 若finally里也有return语句则覆盖try或catch中的return语句直接返回。

**让finally语句不会执行的两种方法**：

1. try语句没有被执行到，如在try语句之前就返回了，这样finally语句就不会执行。

2. 在try块中使用System.exit(0)，终止Java虚拟机JVM。

### 异常类

![img](http://images.cnitblog.com/blog/497634/201402/111228085926220.jpg)

Throwable是Java语言中所有错误或异常的超接口；子类有Error和Exception。

Exception是会被编译器检查的异常(Checked Exception)，**检查性异常**

RuntimeException是Exception的子类，是Java虚拟机正常运行期间抛出的异常的超类，而编译器不会检查RuntimeException异常。 例如：除数为零时，抛出的ArithmeticException异常。**非检查性异常**。

Error：错误，不会被编译器检查。例如：资源不足、约束失败。程序本身无法修复这些错误的。**非检查性异常**。

# 抽象类和接口

接口是对动作的抽象，抽象类是对根源的抽象。

### 抽象类

可以有自己的数据成员。可以有非abstarct的成员方法。可以没有抽象方法。有自己的构造方法。

### 接口

只能有静态的不能被修改的静态的常量（static final）。没有this指针。没有构造方法。JDK1.8中可以使用default关键字实现默认方法和使用静态方法。所有的方法默认是public abstract。

### 注意

1. 抽象类中的抽象方法（加了abstract关键字的方法）不能实现。即不能有{};

#  transient关键字

指定的成员变量在类进行序列化时，不会被序列化。
 (只能修饰变量，不能方法和类，还有本地变量也不可以，静态变量不管是否被transient修饰，均不能被序列化。)

 Serializable接口：所有的序列化将会自动进行，transient关键字有用。

 Externalizable接口：没有任何东西可以自动序列化，需要在writeExternal方法中进行手工指定所要序列化的变量，这与是否被transient修饰无关。

# ==Lambda表达式和方法引用==

### lambda表达式

#### 语法

1. 首先是**参数列表**，这个参数列表与要实现的方法的参数列表一致，用()圆括号括住（当只有一个参数时也可以省略圆括号，但是当没有形参时圆括号不可以省略）（可以省略形参类型）

2. 接下来是一个**->**符号，用于分隔形参列表与函数体，不允许省略。

3. **代码体**，用{}大括号括住。(如果代码体只有一行代码,可以省略掉花括号,如果方法有返回值,连return关键词都可以省略，系统会自动将这一行代码的结果返回。)

lambda表达式的类型必须是函数式接口 Functional Interface，函数式接口代表有且只有一个抽象方法，但是可以包含多个默认方法或类方法的接口。

注释**@FunctionalInterface**用来判别函数式接口。

lambda表达式提供了四种引用方法和构造器的方式：

1. 引用对象的方法 类::实例方法

2. 引用类方法 类::类方法

3. 引用特定对象的方法 特定对象::实例方法

4. 引用类的构造器  类::new
   （系统会自动将函数式接口实现的方法的所有参数中的第一个参数作为调用者，接下来的参数依次传入引用的方法）



### 方法的引用

1. Class::new 构造器引用
2. Class::static_method 静态方法引用
3. Class::method 某个类的成员方法的引用
4. instance::method 某个实例对象的成员方法的引用

# 枚举类

### ==特性==

1. 每一个枚举值是 static final的。
2. 每一个枚举值可以有若干个属性。
3. 每个枚举值拥有各自的内部方法！
4. 枚举类型都隐式继承了`java.lang.Enum`类，因此不能继承其他类，但可以实现接口；
5. 枚举类型只能有私有private的构造方法。
6. 枚举类默认实现了Comparable接口和Serializable接口。
   例如：

```java
public enum Day {
 MONDAY(1, "星期一", "各种不在状态"){
        @Override
        public Day getNext() {
            return TUESDAY;
        }
    },
    TUESDAY(2, "星期二", "依旧犯困"){
        @Override
        public Day getNext() {
            return WEDNESDAY;
        }
    }；
 Day(int index, String name, String value) {
        this.index = index;
        this.name = name;
        this.value = value;
    }

private int index;
private String name;
private String value;
public abstract Day getNext();

。。JAVABEAN方法。。


}
```

### 其他

Enum类重写了Object类的四个方法：finalize，hashCode, clone，equal方法，加了final，不让子类重写：

1. finalize：由于枚举类的实例被static final变量强引用，所以不可被垃圾回收，所以重写没有意义。
2. hashCode：枚举类的实例是不变的，不需要重写。
3. equal：枚举类的实例是不变的，所以直接用==判断是否相等就可以了。
4. clone：枚举类的实例不可被克隆。



# JAVA值传递

### 值传递（pass by value）

是指在调用函数时将==实际参数==复制一份传递到函数中，这样在函数中如果对参数进行修改，将不会影响到实际参数。

### 引用传递（pass by reference）

是指在调用函数时将==实际参数的地址==直接传递到函数中，那么在函数中对参数所进行的修改，将影响到实际参数。

**Java中其实还是值传递的，只不过对于对象参数，值的内容是对象的引用。**

# Java中对象拷贝

浅拷贝(Shallow Copy)、深拷贝(Deep Copy)

### 浅拷贝

对象拷贝时，**基本类型就拷贝的是值，而引用类型拷贝的是内存地址**。即引用变量发生改变了就会相互影响。

### 深拷贝

拷贝时，**直接创建拷贝对象进行拷贝**。即无论是基本类型还是引用对象都互不影响。（可通过序列化实现深拷贝）

# JAVA的基本类型

(1字节=1B=8bit,即8比特位)
byte 1B
short 2B
int 4B
long 8B
float 4B
double 8B
char 2B

---------------------

# Java值得注意的语法

##### ==普通的类方法是可以和类名同名的==

和构造方法唯一的区分就是，构造方法没有返回值。

##### ==与equal的区别

==：对于基本类型，**变量**直接存储的是“值”，比较的就是**值**。而对于**非基本数据类型的变量**，存的是关联的内存地址，因此比较的是**地址**。

equals方法：类中，对没有对equals方法进行重写，比较的是对象的地址，而重写了equals方法的，比较的是对象的内容。

> 基本类型没有equal

##### 线程类型

线程分为守护线程和非守护线程（即用户线程);

只要当前JVM实例中尚存在任何一个非守护线程没有结束，守护线程就全部工作；只有当最后一个非守护线程结束时，守护线程随着JVM一同结束工作。

守护线程最典型的应用就是 GC (垃圾回收器)。

***

 存在使i + 1 < i的数，当i为int能表示的最大整数时，i+1就溢出变成负数了。

 存在使i > j || i <=j不成立的数:比如Double.NaN或Float.NaN

 浮点数后面不写F，默认是double

***

null值可以强制转换为任何java类类型，是合法的。static方法的调用是和类名绑定的，不借助对象进行访问所以能正确输出。

***

##### 对象的初始化顺序

（1）类加载之后，按从上到下（从父类到子类）执行被**static修饰的语句**；

（2）当static语句执行完之后,再执行main方法；

（3）如果有语句new了自身的对象，将从上到下执行**构造代码块{...}**、**构造器**（两者可以说绑定在一起）。

**子类没有显示调用父类构造函数**，不管子类构造函数是否带参数都**默认调用父类无参的构造函数**，若父类没有则编译出错。

***

Object对象的filalize()会在GC回收它时调用。

-----------------------

##### ==方法重载==

方法名字相同，而参数一定要不同。返回类型可以相同也可以不同。可以改变访问修饰符。

***

private修饰的方法与final修饰的方法的区别

private方法：子类还是可以有与父类private修饰的方法同名的方法，两方法各自独立。

final方法：子类不能有与父类final修饰的方法同名的方法。

***

