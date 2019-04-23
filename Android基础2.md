# PackageManager

### 作用

可通过PackageManager类获取指定的App的信息类。

App的信息类包括PackageInfo、ApplicationInfo、ActivityInfo/ServiceInfo/ProviderInfo 等，还有一个 ResolveInfo 类，它比较特殊一点，不和前面的结构为从属关系。

根据 AndroidManifest.xml 中定义的组件进行划分:

![/manifest.png](https://user-gold-cdn.xitu.io/2017/10/19/7de6c948682813fde5e8ad1697fcc0f1?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

一个 PackageInfo 对应一个 ApplicationInfo，而其中又包含若干个 ActivityInfo、ServiceInfo、ProviderInfo。

### PackageManager

是一个抽象类，实际上我们使用的是它的ApplicationPackageManager实现类

获取方式：在Context中，都有`getPackageManager()`方法进行获取。

PackageManager提供很多方法，用于获取PackageInfo对象、ApplicationInfo对象等。

### PackageInfo

主要用于存储获取到的Package的一些信息。

- packageName：包名。
- versionCode：版本号
- versionName：版本名。
- firstInstallTime：首次安装时间。
- lastUpdateTime：最后一次覆盖安装时间。

PackageInfo 中有一个 ApplicationInfo 的字段，是可以直接获取到与它相关的 ApplicationInfo 对象的。

### ApplicationInfo

主要用于获取Apk定义再AndroidManifest.xml的一些信息。

packageName：包名

targetSdkVersion：目标 SDK 版本。

minSdkVersion：最小支持 SDK 版本，有 Api 限制，最低在 Api Level 24 及以上支持。

sourceDir：App 的 Apk 源文件存放的目录。

dataDir：data 目录的全路径。

metaData：Manifest 中定义的 meta 标签数据。

uid：当前 App 分配的 uid。

<https://juejin.im/post/59e87adf51882578c52683ba#heading-7>

# 序列化

### 两接口的区别

- Serializable的本质是使用了反射，序列化的过程比较慢，这种机制在序列化的时候会创建很多临时的对象，比引起频繁的GC、
- Parcelable方式的本质是将一个完整的对象进行分解，而分解后的每一部分都是Intent所支持的类型，这样就实现了传递对象的功能了。

1、如果是仅仅在内存中使用，比如activity、service之间进行对象的传递，强烈推荐使用Parcelable，因为Parcelable比Serializable性能高很多。因为Serializable在序列化的时候会产生大量的临时变量， 从而引起频繁的GC。

2、如果是持久化操作，推荐Serializable，虽然Serializable效率比较低，但是还是要选择它，因为在外界有变化的情况下，Parcelable不能很好的保存数据的持续性。

