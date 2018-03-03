SPI 全称为 (Service Provider Interface) ,是JDK内置的一种服务提供发现机制。

具体 SPI 的介绍详细看下面的资料：

[Java SPI机制简介](http://ivanzhangwb.github.io/blog/2012/06/01/java-spi/)

## Thread.currentThread().getContextClassLoader()

类加载器的简单介绍看如下资料：

[Java 类加载器](http://renchx.com/jvm-classloader/)

线程上下文类加载器（context class loader）是从 JDK 1.2 开始引入的。类 java.lang.Thread 中的方法 getContextClassLoader() 和 setContextClassLoader(ClassLoader cl) 用来获取和设置线程的上下文类加载器。如果没有通过 setContextClassLoader(ClassLoader cl)方法进行设置的话，线程将继承其父线程的上下文类加载器。Java 应用运行的初始线程的上下文类加载器是系统类加载器。在线程中运行的代码可以通过此类加载器来加载类和资源。

### 为什么会有线程上下文类加载器

Thread context class loader 存在的目的主要是为了解决 parent delegation 机制下无法干净的解决的问题。

假如有下述委派链：

ClassLoader A -> System class loader -> Extension class loader -> Bootstrap class loader

那么委派链左边的 ClassLoader 就可以很自然的使用右边的 ClassLoader 所加载的类。

但如果情况要反过来，是右边的 Bootstrap class loader 所加载的代码需要反过来去找委派链靠左边的 ClassLoader A 去加载东西怎么办呢？没辙，parent delegation 是单向的，没办法反过来从右边找左边。

这种情况下就可以把某个位于委派链左边的 ClassLoader 设置为线程的 context class loader，这样就给机会让代码不受 parent delegation 的委派方向的限制而加载到类了。

### 例子

JDBC 接口，我在 classpath 下面引入了 jdbc 的 jar 包。

看如下代码：

```java
public class TestMain {
    public static void main(String[] args) throws Exception {
        Enumeration<Driver> dEnumeration = DriverManager.getDrivers();

        while (dEnumeration.hasMoreElements()) {
            Driver driver = (Driver) dEnumeration.nextElement();
            System.out.println(driver.getClass() + " : " + driver.getClass().getClassLoader());
        }

        System.out.println(Thread.currentThread().getContextClassLoader());
        System.out.println(DriverManager.class.getClassLoader());
    }
}
```

 输出结果如下：

```java
class com.mysql.jdbc.Driver : sun.misc.Launcher$AppClassLoader@2e5f8245
class com.mysql.fabric.jdbc.FabricMySQLDriver : sun.misc.Launcher$AppClassLoader@2e5f8245
sun.misc.Launcher$AppClassLoader@2e5f8245
null
```

我们从结果可以看到，我们并没有注册 Driver 然后就可以获得到 Dirver，并且还可以看到他们的类加载器都是系统加载器。而 DriverManager 的类加载器是 null，也就是说他的加载器是 Bootstrap class loader。

那我们看下 `DriverManager` 源码：

```java
static {
        loadInitialDrivers();
        println("JDBC DriverManager initialized");
    }

private static void loadInitialDrivers() {
        ... 省略
        AccessController.doPrivileged(new PrivilegedAction<Void>() {
            public Void run() {

                ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
                Iterator driversIterator = loadedDrivers.iterator();

                try{
                    while(driversIterator.hasNext()) {
                        driversIterator.next();
                    }
                } catch(Throwable t) {
                // Do nothing
                }
                return null;
            }
        });

        ... 略
    }
```

看主要的部分就是我们使用 SPI 的过程，先加载实现了 Driver 的实现类。

看下 ServiceLoader 是如何进行加载的。

```java
public static <S> ServiceLoader<S> load(Class<S> service) {
        ClassLoader cl = Thread.currentThread().getContextClassLoader();// 使用上下文类加载器，加载器链的逆向使用
        return ServiceLoader.load(service, cl);
    }

public static <S> ServiceLoader<S> load(Class<S> service,
                                            ClassLoader loader)
    {
        return new ServiceLoader<>(service, loader);
    }
```

我们创建了一个 ServiceLoader 实例：

下面是 ServiceLoader 的属性：

```java
  private static final String PREFIX = "META-INF/services/";//SPI 默认读的路径

    // The class or interface representing the service being loaded
    private Class<S> service;//服务的 Class 对象

    // The class loader used to locate, load, and instantiate providers
    private ClassLoader loader;// 类加载器，传入的是线程上下文类加载器

    // Cached providers, in instantiation order
    private LinkedHashMap<String,S> providers = new LinkedHashMap<>(); // 服务的实现者

    // The current lazy-lookup iterator
    private LazyIterator lookupIterator;
```

 下面是构造方法以及 reload 方法：

```java
public void reload() {
        providers.clear();//清空了 map
        lookupIterator = new LazyIterator(service, loader);// 创建 LazyIterator 迭代器
    }

private ServiceLoader(Class<S> svc, ClassLoader cl) {
    service = svc;
    loader = cl;
    reload();// 当创建实例的时候会调用这个 reload 方法。
}
```

看下 LazyIterator 迭代器的实现，从名字就可以看出来是一个懒加载的迭代器。

```java
private class LazyIterator
        implements Iterator<S>
    {

        Class<S> service;
        ClassLoader loader;
        Enumeration<URL> configs = null;
        Iterator<String> pending = null;
        String nextName = null;

        private LazyIterator(Class<S> service, ClassLoader loader) {
            this.service = service;
            this.loader = loader;
        }

        public boolean hasNext() {
            if (nextName != null) {
                return true;
            }
            if (configs == null) {
                try {
                    String fullName = PREFIX + service.getName();
                    if (loader == null)
                        configs = ClassLoader.getSystemResources(fullName);
                    else
                        configs = loader.getResources(fullName);
                } catch (IOException x) {
                    fail(service, "Error locating configuration files", x);
                }
            }
            while ((pending == null) || !pending.hasNext()) {
                if (!configs.hasMoreElements()) {
                    return false;
                }
                pending = parse(service, configs.nextElement());
            }
            nextName = pending.next();
            return true;
        }

        public S next() {
            if (!hasNext()) {
                throw new NoSuchElementException();
            }
            String cn = nextName;
            nextName = null;
            Class<?> c = null;
            try {
                c = Class.forName(cn, false, loader);
            } catch (ClassNotFoundException x) {
                fail(service,
                     "Provider " + cn + " not found");
            }
            if (!service.isAssignableFrom(c)) {
                fail(service,
                     "Provider " + cn  + " not a subtype");
            }
            try {
                S p = service.cast(c.newInstance());
                providers.put(cn, p);
                return p;
            } catch (Throwable x) {
                fail(service,
                     "Provider " + cn + " could not be instantiated",
                     x);
            }
            throw new Error();          // This cannot happen
        }

        public void remove() {
            throw new UnsupportedOperationException();
        }

    }
```

下面看一下 ServiceLoader 的 iterator 方法的实现：

```java
public Iterator<S> iterator() {
        return new Iterator<S>() {

            Iterator<Map.Entry<String,S>> knownProviders
                = providers.entrySet().iterator();

            public boolean hasNext() {
                if (knownProviders.hasNext())
                    return true;
                return lookupIterator.hasNext();
            }

            public S next() {
                if (knownProviders.hasNext())
                    return knownProviders.next().getValue();
                return lookupIterator.next();
            }

            public void remove() {
                throw new UnsupportedOperationException();
            }

        };
    }
```

可以看出来对 `LazyIterator` 进行了一层包装，每次在迭代的时候会把发现的提供者加入到 `ServiceLoader` 内部的一个 map 当中。

回到 JDBC 的那个例子当中，为什么我们会自动进行注册呢，这个需要看一下 `LazyIterator` 的 `next` 方法执行了如下：

```java
c = Class.forName(cn, false, loader); 
```

同时我们在 DriverManager 当中也进行的迭代。最关键的就是 JDBC 规范了，Driver 的实现必须在初始化的时候进行注册。下面看 mysql 里面的实现就清楚了：

```java
static {
        try {
            java.sql.DriverManager.registerDriver(new Driver());
        } catch (SQLException E) {
            throw new RuntimeException("Can't register driver!");
        }
    }
```