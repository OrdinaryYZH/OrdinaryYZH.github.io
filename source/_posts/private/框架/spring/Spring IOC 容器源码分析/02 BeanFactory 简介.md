### BeanFactory的类图以及重要的几个类的作用

![](https://ws1.sinaimg.cn/large/8747d788gy1fy0rttyhrrj21kw0ykwqa.jpg)

这张图呢，背下来肯定是不需要的，有几个重点和大家说明下就好。

1. ApplicationContext 继承了 ListableBeanFactory，这个 Listable 的意思就是，通过这个接口，我们可以获取多个 Bean，大家看源码会发现，最顶层 BeanFactory 接口的方法都是获取单个 Bean 的。
2. ApplicationContext 继承了 HierarchicalBeanFactory，Hierarchical 单词本身已经能说明问题了，也就是说我们可以在应用中起多个 BeanFactory，然后可以将各个 BeanFactory 设置为父子关系。
3. AutowireCapableBeanFactory 这个名字中的 Autowire 大家都非常熟悉，它就是用来自动装配 Bean 用的，但是仔细看上图，ApplicationContext 并没有继承它，不过不用担心，不使用继承，不代表不可以使用组合，如果你看到 ApplicationContext 接口定义中的最后一个方法 getAutowireCapableBeanFactory() 就知道了。
4. ConfigurableListableBeanFactory 也是一个特殊的接口，看图，特殊之处在于它继承了第二层所有的三个接口，而 ApplicationContext 没有。这点之后会用到。
5. 请先不用花时间在其他的接口和类上，先理解我说的这几点就可以了。

### BeanFactory的类图中几个类的大概方法

![](https://ws1.sinaimg.cn/large/8747d788gy1fy0u3dm0gxj21pr1o5anr.jpg)