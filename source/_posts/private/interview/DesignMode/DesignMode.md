### 1. 创建型模式 

#### 1.4 单例模式

单例模式用得最多，错得最多。

1. 饿汉模式最简单：

```java
public class Singleton {
    // 首先，将 new Singleton() 堵死
    private Singleton() {};
    // 创建私有静态实例，意味着这个类第一次使用的时候就会进行创建
    private static Singleton instance = new Singleton();

    public static Singleton getInstance() {
        return instance;
    }
    // 瞎写一个静态方法。这里想说的是，如果我们只是要调用 Singleton.getDate(...)，
    // 本来是不想要生成 Singleton 实例的，不过没办法，已经生成了
    public static Date getDate(String mode) {return new Date();}
}

```

> 优点：依赖 JVM 在类装载时就完成唯一对象的实例化，基于类加载的机制，它们**天生就是线程安全的**
>
> 很多人都能说出饿汉模式的缺点，可是我觉得生产过程中，很少碰到这种情况：你定义了一个单例的类，不需要其实例，可是你却把一个或几个你会用到的静态方法塞到这个类中。

2. 饱汉模式最容易出错：

```java
public class Singleton {
    // 首先，也是先堵死 new Singleton() 这条路
    private Singleton() {}
    // 和饿汉模式相比，这边不需要先实例化出来，注意这里的 volatile，它是必须的
    private static volatile Singleton instance = null;

    public static Singleton getInstance() {
        // 第一次check，避免不必要的同步
        if (instance == null) {
            // 加锁
            synchronized (Singleton.class) {
                // 这一次判断也是必须的，不然会有并发问题
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}

```

> 双重检查，指的是两次检查 instance 是否为 null。
>
> volatile 在这里是需要的，希望能引起读者的关注。
>
> 很多人不知道怎么写，直接就在 getInstance() 方法签名上加上 synchronized，这就不多说了，性能太差。

3. 嵌套类最经典，以后大家就用它吧：

```java
public class Singleton3 {

    private Singleton3() {}

    public static Singleton3 getInstance() {
        return Holder.instance;
    }
  
    private static class Holder {
        private static Singleton3 instance = new Singleton3();
    }
}

```

> 注意，很多人都会把这个**嵌套类**说成是**静态内部类**，严格地说，内部类和嵌套类是不一样的，它们能访问的外部类权限也是不一样的。
>
> **这种方式利用了 classloder 的机制来保证初始化 instance 时只会有一个。需要注意的是：虽然它的名字中有“静态”两字，但它是属于“懒汉模式”的！！这种方式的 Singleton 类被装载时，只要 SingletonHolder 类还没有被主动使用，instance 就不会被初始化。只有在显式调用 getInstance() 方法时，才会装载 SingletonHolder 类，从而实例化对象。**
>
> 参考：[【J2SE】为什么静态内部类的单例可以实现延迟加载](http://blog.csdn.net/reliveit/article/details/52874833)

最后，一定有人跳出来说用枚举实现单例，是的没错，枚举类很特殊，它在类加载的时候会初始化里面的所有的实例，而且 JVM 保证了它们不会再被实例化，所以它天生就是单例的。不说了，读者自己看着办吧，不建议使用。

### 2. 结构型模式

### 3.行为型模式

