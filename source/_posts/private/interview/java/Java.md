

# Java

## 面向对象

1. 三大特性

   封装、继承、多态

## 类的概念

### 理解接口，抽象类，内部类，包，枚举等本质

#### 内部类的本质 

1. 内部类相关知识点

## 线程

1. Java线程的状态
   [Java 线程的七种状态](http://haohaochang.cn/2017/07/21/Java-thread-7-kinds-of-state/)
   ![](https://ws1.sinaimg.cn/large/8747d788ly1fplmndxov6j21sa17lan6.jpg)

2. 进程与线程的区别，进程间如何通讯，线程间如何通讯？

3. 写出java中synchronized的使用方式

   ```java
   public class LockStrategy
   {
       public Object object1 = new Object();

       public static synchronized void method1(){}
       public void method2(){
           synchronized(LockStrategy.class){}
       }

       public synchronized void method4(){}
       public void method5()
       {
           synchronized(this){}
       }
       public void method6()
       {
           synchronized(object1){}
       }
   }
   ```

4. volatile的作用？
    Java 语言中的 volatile 变量可以被看作是一种 “程度较轻的 `synchronized`
       ”锁提供了两种主要特性：*互斥（mutual exclusion）* 和*可见性（visibility）*

   > 互斥即一次只允许一个线程持有某个特定的锁，因此可使用该特性实现对共享数据的协调访问协议，这样，一次就只有一个线程能够使用该共享数据。可见性要更加复杂一些，它必须确保释放锁之前对共享数据做出的更改对于随后获得该锁的另一个线程是可见的 —— 如果没有同步机制提供的这种可见性保证，线程看到的共享变量可能是修改前的值或不一致的值，这将引发许多严重问题。

   参考：[正确使用 Volatile 变量](https://www.ibm.com/developerworks/cn/java/j-jtp06197.html)

## 容器

1. HashMap的数据结构是什么？如何实现的？和HashTable、ConcurrentHashMap的区别？
   Java7：数组 + 链表
   Java8：数组 + 链表 + 红黑树
   HashMap线程不安全，HashTable、ConcurrentHashMap线程安全
   HashMap能够添加key为null，HashTable不能

2. ArrayList是如何实现的，ArrayList和LinkedList的区别？ArrayList如何实现扩容？

   * 就是数组啊；
   * ArrayList：
     * 按数组下标访问元素－get（i）、set（i,e） 的性能很高，这是数组的基本优势。
     * 如果按下标插入元素、删除元素：add（i,e）、 remove（i）、remove（e），则要用System.arraycopy（）来复制移动部分受影响的元素，性能就变差了。
   * LinkedList
     * 按下标访问元素－get（i）、set（i,e） 要悲剧的部分遍历链表将指针移动到位 （如果i>数组大小的一半，会从末尾移起）。
     * 插入、删除元素时修改前后节点的指针即可，不再需要复制移动。但还是要部分遍历链表的指针才能移动到下标所指的位置。
   * 扩容：超出限制时会增加50%容量，用System.arraycopy（）复制到新的数组

3. 谈谈你对HashMap的理解，怎么样去保证线程安全？

   * 是啥：【key，value】集合

   * 如何做：hash + 数组 + 链表/红黑树

   * 三种做法，但ConcurrentHashMap效率高

     ```java
     //Hashtable
     Map<String, String> normalMap = new Hashtable<>();

     //synchronizedMap
     Map<String, String> synchronizedHashMap = Collections.synchronizedMap(new HashMap<String, String>());

     //ConcurrentHashMap
     ConcurrentHashMap<String, String> concurrentHashMap = new ConcurrentHashMap<>();
     ```

     ​

4. Java集合中有哪些常用的类？ArrayList的上级（父类或者接口）是什么，HashMap的上级又是什么？ 

   1. List：

      - ArrayList
      - LinkedList
      - CopyOnWriteArrayList
   2. Map：
      - HashMap
      - LinkedHashMap
      - TreeMap
      - ConcurrentHashMap
   3. Set：
      - HashSet
      - LinkedHashSet
      - TreeSet
      - CopyOnWriteArraySet
      - ConcurrentHashSet？不存在的，使用：Collections.newSetFromMap(new ConcurrentHashMap())
   4. Queue：

      - 普通队列：

        - LinkedList
        - ArrayDeque
        - PriorityQueue
      - 线程安全的队列

        - ConcurrentLinkedQueue/Deque
      - 线程安全的阻塞队列

        - ArrayBlockingQueue
        - LinkedBlockingQueue/Deque
        - PriorityBlockingQueue
        - DelayQueue
      - 同步队列
        - SynchronousQueue
   5. ArrayList的上级（父类或者接口）:
      ![](https://ws1.sinaimg.cn/large/8747d788gy1fpye82vb74j20zj0hata2.jpg)
   6. HashMap的上级:
      ![](https://ws1.sinaimg.cn/large/8747d788gy1fpye8nhlppj20n00ay3z3.jpg)

## IO

1. nio相关知识点
2. 一句话概括NIO

## 其他（不知道如何分类）

1. equals、hashcode等Object类中一些方法的讨论？
   [第9条：覆盖equals时总要覆盖hashCode](https://zhengyq.gitbooks.io/effective-java-2/content/item9.html)