# ClassLoader

## 什么是ClassLoader

> Java 代码要想运行，首先需要将源代码进行编译生成 .class 文件，然后 JVM 加载 .class 字节码文件到内存，而 .class 文件是怎样被加载到 JVM 中的就是Java ClassLoader 要做的事情。

### .class 文件何时被类加载器加载到 JVM 中？

> 1.  new 操作时候；
>
> 2. 当我们使用 Class.forName("包路径+类名")、Class.forName("包路径+类名",ClassLoader)
>
> 3. ClassLoader.loadClass("包路径+类名")的时候就触发了类加载器去类加载对应的路径去查找 *.class，并创建 Class 对象。
>
>    ▲注意：另外需要注意的是除去 new 操作外，其他几种方式加载字节码到内存后只是生产一个 Class 对象，要产生具体的对象实例还需要使用 Class 对象 .newInstance() 函数来创建。

## Java 原生的三种 ClassLoader

### AppClassloader（应用类加载器/系统类加载器）

> 它负责在 JVM 启动时，加载来自在命令java中的-classpath或者java.class.path系统属性或者 CLASSPATH 操作系统属性所指定的 JAR 类包和类路径

1. API:
   静态方法`ClassLoader.getSystemClassLoader()`：可以获得AppClassLoader
2. 如果没有特别指定，则用户自定义的任何类加载器都将该类加载器作为它的父加载器，这点通过 `java.lang.ClassLoader` 的无参构造函数可以证明，代码如下:
```java
protected ClassLoader() {
    this(checkCreateClassLoader(),getSystemClassLoader());
}
```

3. 获取classpath加载路径：
   `System.out.println(System.getProperty("java.class.path"));`
4. 另外我们写的含有 main 函数的类的加载就是使用该类加载器进行加载的，证明如下：
```java
public static void main(String[] args) {
    ClassLoader cl = App2.class.getClassLoader();
    System.out.println(cl);
    System.out.println(cl.getParent());
}
```

运行结果如下。

```java
sun.misc.Launcher$AppClassLoader@4554617c
sun.misc.Launcher$ExtClassLoader@677327b6
```

并且可以看出 AppClassLoader 的父加载器是 ExtClassLoader，那么 ExtClassLoader 是什么呢？

### ExtClassloader

> 扩展类加载器，主要负责加载 Java 的扩展类库

1. 加载的路径
   默认加载 `JAVA_HOME/jre/lib/ext/` 目录下的所有 Jar 包或者由 `java.ext.dirs` 系统属性指定的 Jar 包
   可以通过系统属性`java.ext.dirs`查看
2. ExtClassLoader 的父加载器为 null

### BootstrapClassloader

> 引导类加载器，又称启动类加载器，是最顶层的类加载器，主要用来加载 Java 核心类，如 rt.jar、resources.jar、charsets.jar 等。
>
> 需要注意的是它不是 java.lang.ClassLoader 的子类，而是由 JVM 自身实现的，该类为 C 语言实现，所以严格来说它不属于 Java 类加载器范畴，Java 程序访问不到该加载器。

通过下面代码我们可以查看该加载器查找类的扫描路径。

```java
public void test() {
	URL[] urls = sun.misc.Launcher.getBootstrapClassPath().getURLs();
	for (int i = 0; i < urls.length; i++) {
		System.out.println(urls[i].toExternalForm());
    }
}
```

执行结果如下。

![](https://ws1.sinaimg.cn/large/8747d788gy1foxqct6w91j21ub0g21kx.jpg)

`String` 类就是 `rt.jar` 里面提供的，这个类我们经常用，下面我们看下 String 类的类加载器是什么。

```java
    public static void main(String[] args) {    
        ClassLoader cl = String.class.getClassLoader();
        System.out.println(cl);
    }
```

执行结果如下。

```
null
```

可知由于 BootstrapClassloader 对 Java 不可见，所以返回了 null，我们也可以通过看某一个类的加载器是否为 null 来作为判断该类是不是使用 BootstrapClassloader 进行加载的依据。另外上面提到 ExtClassLoader 的 父加载器返回的 null，那是否说明 ExtClassLoader 的父加载器是 BootstrapClassloader呢？

### 三种加载器关系

首先用一张图来表示三张图的关系如下：

![](https://ws1.sinaimg.cn/large/8747d788gy1fv98wlsmnlj20r20pqadq.jpg)

- AppClassloader 的父加载器是 ExtClassloader。
- ExtClassloader 的父加载器为 null，但是要注意的是 ExtClassloader 的父加载器并不是 BootstrapClassloader。

### 类加载器的构造

下面从源码来分析下 JVM 是如何构建内置 Classloader 的，具体是 rt.jar 包里面`sun.misc.Launcher`类。代码如下。

```java
public Launcher()  
      {  
        ExtClassLoader localExtClassLoader;  
        try  
        {  //（1）首先创建了ExtClassLoader
          localExtClassLoader = ExtClassLoader.getExtClassLoader();  
        }  
        catch (IOException localIOException1)  
        {  
          throw new InternalError("Could not create extension class loader");  
        }  
        try  
        {  //（2）然后以ExtClassloader作为父加载器创建了AppClassLoader
          this.loader = AppClassLoader.getAppClassLoader(localExtClassLoader);  
        }  
        catch (IOException localIOException2)  
        {  
          throw new InternalError("Could not create application class loader");  
        }  //（3）这个是个特殊的加载器后面会讲到，这里只需要知道默认下线程上下文加载器为appclassloader
        Thread.currentThread().setContextClassLoader(this.loader);  

        ................
     
```

代码（1）首先创建了 `ExtClassLoader` 类加载器，下面我们看看具体创建过程，打开 `ExtClassLoader.getExtClassLoader()` 的代码，如下。

```java
public static ExtClassLoader getExtClassLoader()  
      throws IOException  
    {  
      File[] arrayOfFile = getExtDirs();  
      try  
      {  
        (ExtClassLoader)AccessController.doPrivileged(new PrivilegedExceptionAction()  
        {  
          public Launcher.ExtClassLoader run()  
            throws IOException  
          {  
            int i = this.val$dirs.length;  
            for (int j = 0; j < i; j++) {  
              MetaIndex.registerDirectory(this.val$dirs[j]);  
            }  
            //(5)
            return new Launcher.ExtClassLoader(this.val$dirs);  
          }  
        });  
      }  
      catch (PrivilegedActionException localPrivilegedActionException)  
      {  
        throw ((IOException)localPrivilegedActionException.getException());  
      }  
    }  

   //(6)
    private static File[] getExtDirs()  
    {  
      String str = System.getProperty("java.ext.dirs");  
      File[] arrayOfFile;  
      if (str != null)  
      {  
        StringTokenizer localStringTokenizer = new StringTokenizer(str, File.pathSeparator);  

        int i = localStringTokenizer.countTokens();  
        arrayOfFile = new File[i];  
        for (int j = 0; j < i; j++) {  
          arrayOfFile[j] = new File(localStringTokenizer.nextToken());  
        }  
      }  
      else  
      {  
        arrayOfFile = new File[0];  
      }  
      return arrayOfFile;  
    }  
```

从代码（6）可知，`ExtClassLoader` 类加载类扫描路径为 `java.ext.dirs`。下面我们看看代码（5），它说明 `ExtClassLoader` 的父加载器为 null，打开 `Launcher.ExtClassLoader` 的代码如下。

```java
   public ExtClassLoader(File[] paramArrayOfFile)
      throws IOException
    {
     //(7)第一个参数，就是父加载器的设置，这里传递了null。
      super(null, Launcher.factory);
      SharedSecrets.getJavaNetAccess()
        .getURLClassPath(this).initLookupCache(this);
    }
```

代码（2）以创建的` ExtClassloader` 作为父加载器创建了 `AppClassLoader`，下面看下 `AppClassLoader.getAppClassLoader` 的代码，如下。

```java
public static ClassLoader getAppClassLoader(final ClassLoader paramClassLoader)  
      throws IOException  
    {  //(8)
      String str = System.getProperty("java.class.path");  
      final File[] arrayOfFile = str == null ? new File[0] : Launcher.getClassPath(str);  

      (ClassLoader)AccessController.doPrivileged(new PrivilegedAction()  
      {  
        public Launcher.AppClassLoader run()  
        {  
          URL[] arrayOfURL = this.val$s == null ? new URL[0] : Launcher.pathToURLs(arrayOfFile); 
          return new Launcher.AppClassLoader(arrayOfURL, paramClassLoader);  
        }  
      });  
    }  
```

由代码（8）可知 `AppClassLoader` 类加载扫描路径为为 `java.class.path`。下面看下 AppClassLoader 的父加载器的设置，看下 `Launcher.AppClassLoader` 的代，如下。

```java
 AppClassLoader(URL[] paramArrayOfURL, ClassLoader paramClassLoader)
    {
     //paramClassLoader就是ExtClassloader
      super(paramClassLoader, Launcher.factory);
      this.ucp = SharedSecrets.getJavaNetAccess().getURLClassPath(this);
      this.ucp.initLookupCache(this);
    }
```

代码（3）创建了一个与线程相关的类加载器，这个后面会讲到。

### 类加载器原理

> Java 类加载器使用的是委托机制，也就是一个类加载器在加载一个类时候会首先尝试让父类加载器来加载。

那么问题来了，为啥使用这种方式？

> 第一，这样可以避免重复加载，当父类加载器已经加载了该类的时候，就没有必要子 ClassLoader 再加载一次。
>
> 第二，考虑到安全因素，我们试想一下，如果不使用这种委托模式，那我们就可以随便使用自定义的类来动态替代 Java 核心 API 中类实现，比如我们自己写了个 String 类，包路径+类名与 rt.jar 里面的一样，如果不用委托机制，那么当 JVM 加载 String 类的时候会使用 AppClassLoader 加载我们自己定义的 String 类而不会去加载 rt.jar 里面的了。使用双亲委托则，当 JVM 加载 String 类的时候， AppClassLoader 会委托父类加载器 ExtClassLoader 来加载，ExtClassLoader 又会委托给 Bootstrcp ClassLoader 来加载，这样就不会加载自定义的 String 类了。

下面我们从源码看如何实现委托机制。

```java
protected Class<?> loadClass(String name,boolean resolve)  
       throws ClassNotFoundException  
   {  
       synchronized (getClassLoadingLock(name)) {  
           // 首先从jvm缓存查找该类
           Class c = findLoadedClass(name); // (1)
           if (c ==null) {  
               longt0 = System.nanoTime();  
               try {  //然后委托给父类加载器进行加载
                   if (parent !=null) {  
                       c = parent.loadClass(name,false);  (2)
                   } else {  //如果父类加载器为null,则委托给BootStrap加载器加载
                       c = findBootstrapClassOrNull(name);  (3)
                   }  
               } catch (ClassNotFoundExceptione) {  
                   // ClassNotFoundException thrown if class not found  
                   // from the non-null parent class loader  
               }  

               if (c ==null) {  
                   // 若仍然没有找到则调用findclass查找
                   // to find the class.  
                   longt1 = System.nanoTime();  
                   c = findClass(name);  (4)

                   // this is the defining class loader; record the stats  
                   sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 -t0);  
                   sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);  
                   sun.misc.PerfCounter.getFindClasses().increment();  
               }  
           }  
           if (resolve) {  
               resolveClass(c);  //（5）
           }  
           returnc;  
       }  
   }  
```

代码（1）表示从 JVM 缓存查找该类，如果该类之前被加载过，则直接从 JVM 缓存返回该类。

代码（2）表示如果 JVM 缓存不存在该类，则看当前类加载器是否有父加载器，如果有的话则委托父类加载器进行加载，否者调用（3），委托 BootStrapClassloader 进行加载，如果还是没有找到，则调用当前 Classloader 的 findclass 方法进行查找。

代码（5）则是当字节码加载到内存后进行链接操作，对文件格式和字节码验证，并为 static 字段分配空间并初始化，符号引用转为直接引用，访问控制，方法覆盖等，本文对这些不进入深入探讨。

从上面源码知道要想修改类加载委托机制，实现自己的载入策略，可以通过覆盖 ClassLoader 的 findClass 方法或者覆盖 loadClass 方法来实现。

> ▲需要了解的是，"双亲委派"虽然是一般模型，但也有一些例外，比如：
>
> - 自定义的加载顺序：尽管不被建议，自定义的ClassLoader可以不遵从"双亲委派"这个约定，不过，即使不遵从，以"java"开头的类也不能被自定义类加载器加载，这是由Java的安全机制保证的，以避免混乱。
> - 网状加载顺序：在OSGI框架中，类加载器之间的关系是一个网，每个OSGI模块有一个类加载器，不同模块之间可能有依赖关系，在一个模块加载一个类时，可能是从自己模块加载，也可能是委派给其他模块的类加载器加载。
> - 父加载器委派给子加载器加载：典型的例子有JNDI服务(Java Naming and Directory Interface)，它是Java企业级应用中的一项服务，具体我们就不介绍了。

### ClassLoader vs Class.forName

在[反射](http://mp.weixin.qq.com/s?__biz=MzIxOTI1NTk5Nw==&mid=2650047510&idx=1&sn=6d8873ffbfd31e802d87ebf4ac135f39&chksm=8fde21c4b8a9a8d2df3eaec9a88ec9dfe748b2bfe7066cea0ea7c3dfc4f1f37d559305cccf7c&scene=21#wechat_redirect)一节，我们介绍过Class的两个静态方法forName：

```java
public static Class<?> forName(String className)
public static Class<?> forName(String name, boolean initialize, ClassLoader loader)
```

第一个方法使用系统类加载器加载，第二个指定`ClassLoader`，参数`initialize`表示，加载后，是否执行类的初始化代码(如static语句块)，没有指定默认为true。

`ClassLoader`的`loadClass`方法与`forName`方法都可以加载类，它们有什么不同呢？

基本是一样的，不过，有一个不同，**`ClassLoader`的`loadClass`不会执行类的初始化代码**，看个例子：

```java
public class CLInitDemo {
    public static class Hello {
        static {
            System.out.println("hello");
        }
    };

    public static void main(String[] args) {
        ClassLoader cl = ClassLoader.getSystemClassLoader();
        String className = CLInitDemo.class.getName() + "$Hello";
        try {
            Class<?> cls = cl.loadClass(className);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

使用ClassLoader加载静态内部类Hello，Hello有一个static语句块，输出"hello"，运行该程序，类被加载了，但没有任何输出，即static语句块没有被执行。如果将loadClass的语句换为：

```java
Class<?> cls = Class.forName(className);
```

则static语句块会被执行，屏幕将输出"hello"。



## 一种特殊的类加载器：ContextClassLoader

> `ContextClassLoader` 是一种与线程相关的类加载器，类似 `ThreadLocal`，每个线程对应一个上下文类加载器：
>
> ```java
>     /* The context ClassLoader for this thread */
>     private ClassLoader contextClassLoader;
> ```
>
> 在使用时，一般都用下面的经典结构。
>
> ```java
> //获取当前线程上下文类加载器
> ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
> try {//设置当前线程上下文类加载器为targetTccl
>     Thread.currentThread().setContextClassLoader(targetTccl);
>     //doSomething 
>     doSomething();
> } finally {//设置当前线程上下文加载器为原始加载器
>     Thread.currentThread().setContextClassLoader(classLoader);
> }
> ```
>
> 首先获取当前线程的线程上下文类加载器并保存到方法栈，然后设置当前线程上下文类加器为自己的类加载器。
>
> `doSomething` 里面则调用了 `Thread.currentThread().getContextClassLoader()`，获取当前线程上下文类加载器做某些事情。
>
> 最后在设置当前线程上下文类加载器为老的类加载器。

那么这其中的奥秘和使用场景是什么？我们知道 Java 默认的类加载机制是委托机制，但是有些时候需要破坏这种机制。

具体来说，比如 Java 中的 SPI（Service Provider Interface）是面向接口编程的，服务规则提供者会在 JRE 的核心 API 里面提供服务访问接口，而具体实现则由其他开发商提供。我们知道 Java 核心 API，比如rt.jar包，是使用 `Bootstrap CalssLoader` 加载的，而用户提供的 Jar 包再由 `Appclassloader` 加载。并且我们知道如果一个类由类加载器 A 加载，那么这个类依赖类也是由相同的类加载器加载。那么 Bootstrap ClassLoader 加载了服务提供者在 rt.jar 里面提供的搜索开发商提供的实现类的 API 类（`ServiceLoader`），那么这些 API 类里面依赖的类应该也是由 `Bootstrap CalssLoader` 来加载。而上面说了用户提供的 Jar 包再由 `Appclassloader` 加载，所以需要一种违反双亲委派模型的方法，线程上下文类加载器就是为了解决这个问题。

下面使用 JDBC 4 来具体说明，JDBC 4 是基于 SPI 机制来发现驱动提供商提供的实现类，提供者只需在 JDBC 实现的 Jar 的 `META-INF/services/java.sql.Driver` 文件里指定实现类的方式暴露驱动提供者。例如 MySQL 实现的 Jar，如下。

![screenshot.png](http://upload-images.jianshu.io/upload_images/5879294-5bc66b9ae0be8415.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

而 MySQL 的驱动如下实现了 java.sql.Driver。

```java
public class com.mysql.jdbc.Driver extends com.mysql.jdbc.NonRegisteringDriver implements java.sql.Driver
```

OK，下面我们写个测试代码，看看具体是如何工作的。

```java
    public static void main(String[] args) {
               //(1)
        ServiceLoader<Driver> loader = ServiceLoader.load(Driver.class);
               //(2)
        Iterator<Driver> iterator = loader.iterator();
        while (iterator.hasNext()) {
            Driver driver = (Driver) iterator.next();
            System.out.println("driver:" + driver.getClass() + ",loader:" + driver.getClass().getClassLoader());
        }
        //(3)
        System.out.println("current thread contextloader:" + Thread.currentThread().getContextClassLoader());
               //(4)
        System.out.println("ServiceLoader loader:" + ServiceLoader.class.getClassLoader());
    }
}
```

然后引入 MySQL 驱动的 Jar 包，执行结果如下。

```java
driver:class com.mysql.jdbc.Driver,loader:sun.misc.Launcher$AppClassLoader@4554617c
current thread contextloader:sun.misc.Launcher$AppClassLoader@4554617c
ServiceLoader loader:null
```

从执行结果可以知道 ServiceLoader 的加载器为 Bootstarp，因为这里输出了 null，并且从该类在 rt.jar 里面，也可以证明。

当前线程上下文类加载器为 AppClassLoader。而 com.mysql.jdbc.Driver 则使用 AppClassLoader 加载。我们知道如果一个类中引用了另外一个类，那么被引用的类也应该由引用方类加载器来加载，而现在则是引用方 ServiceLoader 使用 BootStarpClassloader 加载，被引用方则使用子加载器 APPClassLoader 来加载了，是不是很诡异。

下面我们来看下 ServiceLoader 的 load 方法源码。

```java
public final class ServiceLoader<S> implements Iterable<S> {
    public static <S> ServiceLoader<S> load(Class<S> service) {
        // （5）获取当前线程上下文加载器
        ClassLoader cl = Thread.currentThread().getContextClassLoader();
        return ServiceLoader.load(service, cl);
    }

    public static <S> ServiceLoader<S> load(Class<S> service, ClassLoader loader) {
        return new ServiceLoader<>(service, loader);
    }
       //(6)
    private ServiceLoader(Class<S> svc, ClassLoader cl) {
        service = svc;
        loader = cl;
        reload();
    }
```

代码（5）获取了当前线程上下文加载器，这里是 AppClassLoader。

代码（6）传递该类加载器到新构造的 ServiceLoader 的成员变量 loader。那么这个 loader 什么时候使用的呢？下面我们看下 LazyIterator 的 next() 方法。

```java
          public S next() {
            if (acc == null) {
                return nextService();
            } else {
                PrivilegedAction<S> action = new PrivilegedAction<S>() {
                    public S run() { return nextService(); }
                };
                return AccessController.doPrivileged(action, acc);
            }
        }
```

```java
  private S nextService() {
   ...
    try {
        //（7）使用loader类加载器加载
        c = Class.forName(cn, false, loader);
    } catch (ClassNotFoundException x) {
        fail(service,
             "Provider " + cn + " not found");
    }
   ...
    }
```

代码（7）使用 loader 也就是 AppClassLoader 加载具体的驱动实现类。至于 cn 是怎么来的，读者可以参见 LazyIterator 的 hasNext() 方法。

到这里再回想下，ContextClassLoader 的作用是为了破坏 Java 类加载委托机制，JDBC 规范定义了一个 JDBC 接口，然后使用 SPI 机制提供的一个叫做 ServiceLoader 的 Java 核心 API（rt.jar 里面提供）用来扫描服务实现类，服务实现者提供的 Jar，比如 MySQL 驱动则是放到我们的 classpath 下面，从上文知道默认线程上下文类加载器就是 AppClassLoader，所以例子里面没有显示在调用 ServiceLoader 前设置线程上下文类加载器为 AppClassLoader，ServiceLoader 内部则获取当前线程上下文类加载器（这里为 AppClassLoader）来加载服务实现者的类，这里加载了 classpath 下的MySQL 的驱动实现。

读者可以尝试在调用 ServiceLoader 的 load 方法前设置线程上下文类加器为 ExtclassLoder，代码如下：

```java
Thread.currentThread().setContextClassLoader(main函数所在的类.class.getClassLoader().getParent());
```

然后再运行本例子，设置后 ServiceLoader 内部则获取当前线程上下文类加载器为 ExtclassLoder，然后会尝试使用 ExtclassLoder 去查找 JDBC 驱动实现，而 ExtclassLoder 扫描类的路径为 JAVA_HOME/jre/lib/ext/，而这下面没有驱动实现的 Jar，所以不会查找到驱动。

总结下，**当父类加载器需要加载子类加载器中的资源时，可以通过设置和获取线程上下文类加载器来实现**。

---

个人总结：

这里以大概跟踪了ServiceLoader的代码，讲了如何利用ContextClassLoader；

`ServiceLoader`的作用：使用线程传过来的ClassLoader来加载Class

```java
public static <S> ServiceLoader<S> load(Class<S> service) {
    ClassLoader cl = Thread.currentThread().getContextClassLoader();
    return ServiceLoader.load(service, cl);
}
```

```java
public static <S> ServiceLoader<S> load(Class<S> service,
                                        ClassLoader loader) {
    return new ServiceLoader<>(service, loader);
}
```

```java
private ServiceLoader(Class<S> svc, ClassLoader cl) {
    service = Objects.requireNonNull(svc, "Service interface cannot be null");
    loader = (cl == null) ? ClassLoader.getSystemClassLoader() : cl;
    acc = (System.getSecurityManager() != null) ? AccessController.getContext() : null;
    reload();
}
```

[关于ContextClassLoader的另一篇文章](http://alicharles.com/article/java-spi-serviceloader/)



## Tomcat ClassLoader（略）

## 使用自定义类加载器实现模块隔离

## 自定义ClassLoader的应用 - 热部署

参考：[Java编程的逻辑 (87) - 类加载机制](http://www.cnblogs.com/swiftma/p/6901301.html)





