## IOC

##### 1. BeanFactory和ApplicationContext的区别

ApplicationContext 继承自 BeanFactory，但是它不应该被理解为 BeanFactory 的实现类，而是说其内部持有一个实例化的 BeanFactory（DefaultListableBeanFactory）。以后所有的 BeanFactory 相关的操作其实是给这个实例来处理的

![](https://ws1.sinaimg.cn/large/8747d788gy1fpsk0pdh60j20wm0nd42m.jpg)

##### 2. Spring IOC的原理？

   答案：基于反射

## AOP

![](https://ws1.sinaimg.cn/large/8747d788gy1fplferfwf5j20w40hzabd.jpg)

##### 1. 事务实现原理
   基于Spring AOP
   [透彻的掌握 Spring 中@transactional 的使用](https://www.ibm.com/developerworks/cn/java/j-master-spring-transactional-use/index.html)

##### 2. spring事务传播特性

   * REQUIRED
   * REQUIRES_NEW
   * MANDATORY
   * NESTED
   * SUPPORTS
   * NOT_SUPPORTED
   * NEVER

   [一图学习 Spring事务传播性](http://yhzhtk.info/2014/06/17/mindmap-transaction-propagation.html)

##### 3. Spring Aop的原理
   动态代理，实现方式有JDK动态代理和CGLIB

## 其他

##### 1. Spring常用的注解

1. 组件类注解
  * @Component ：标准一个普通的spring Bean类。 
  * @Repository：标注一个DAO组件类。 
  * @Service：标注一个业务逻辑组件类。 
  * @Controller：标注一个控制器组件类。
2. 装配bean时常用的注解
   * @Autowired：属于Spring 的org.springframework.beans.factory.annotation包下,可用于为类的属性、构造器、方法进行注值 
   * @Resource：不属于spring的注解，而是来自于JSR-250位于java.annotation包下，使用该annotation为目标bean指定协作者Bean。 
   * @PostConstruct 和 @PreDestroy 方法 实现初始化和销毁bean之前进行的操作
3. @Configuration and @Bean
4. spring MVC模块注解
   * @Controller
   * @RequestMapping、@GetMapping、@PostMapping...
   * @RequestParam
   * @PathVariable
   * @RequestBody
   * @ResponseBody
   * @RestController
   * @ControllerAdvice
5. Spring事务模块注解
   * @Transactional

   其他参考：http://www.rowkey.me/blog/2017/10/28/spring-annotations/


##### 2.  Spring中的注解在xml文件中怎么配置？

   ```xml
<context:annotation-config>
   ```
##### 3. Spring中的自动装配怎么配置？

   ```xml
<context:componenet-scan>
   ```

