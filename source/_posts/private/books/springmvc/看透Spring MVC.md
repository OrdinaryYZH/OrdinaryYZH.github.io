## 第九章 创建Spring MVC之器

### 9.2 HttpServletBean

`init()`方法如下：

```java
@Override
public final void init() throws ServletException {
   if (logger.isDebugEnabled()) {
      logger.debug("Initializing servlet '" + getServletName() + "'");
   }

   // Set bean properties from init parameters.
   PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties);
   if (!pvs.isEmpty()) {
      try {
         BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
         ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext());
         bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, getEnvironment()));
         initBeanWrapper(bw);	// 模板方法，也没子类实现
         bw.setPropertyValues(pvs, true);
      }
      catch (BeansException ex) {
         if (logger.isErrorEnabled()) {
            logger.error("Failed to set bean properties on servlet '" + getServletName() + "'", ex);
         }
         throw ex;
      }
   }

   // Let subclasses do whatever initialization they like.
   initServletBean();	// 模板方法

   if (logger.isDebugEnabled()) {
      logger.debug("Servlet '" + getServletName() + "' configured successfully");
   }
}
```

> **HttpServletBean做了什么？**
>
> 首先将Servlet中配置的参数使用BeanWrapper设置到DispatcherServlet的相关属性，然后调用模板方法initServletBean，子类就通过这个方法初始化 

需要的知识点：

1. **BeanWrapper**
   使用例子：
   ![](http://ww1.sinaimg.cn/large/8747d788gy1ftyy3x8addj21h10t0154.jpg)
2. **PropertyValue/PropertyValues**
   PropertyValues类其实是一个容器类，它内部会维护一个PropertyValue (注意此处没有s) 的数组，而PropertyValue是一个专门存放属性／参数的结构，它可以存储一个String类型的属性名以及一个Object类型的属性对象。 
   参考：[spring笔记-AttributeAccessor](https://www.jianshu.com/p/3b338dda2437)
3. **ResourceLoader**：
   这个接口定义了一些获取Resource的方法，你可以用它来获取各种各样的资源，比如配置文件，class文件等。那ServletContextResourceLoader的实现其实是提供了获取那些和当前Servlet相关的一些Resource，它会通过ServletContext中的一些方法去获取资源，比如servletContext.getResourceAsStream(), servletContext.getResource()
4. **ResourceEditor**
   Spring提供了一个PropertyEditor “ResourceEditor”用于在注入的字符串和Resource之间进行转换   



### 9.3 FrameworkServlet

FrameworkServlet的初始化入口方法是`initServletBean()`，这里只做了2件事：

1. 初始化WebApplicationContext
2. 初始化FrameworkServlet：initFrameworkServlet()是模板方法，但是没子类实现

代码如下：

```java
@Override
protected final void initServletBean() throws ServletException {
   getServletContext().log("Initializing Spring FrameworkServlet '" + getServletName() + "'");
   if (this.logger.isInfoEnabled()) {
      this.logger.info("FrameworkServlet '" + getServletName() + "': initialization started");
   }
   long startTime = System.currentTimeMillis();

   try {
1.--→ this.webApplicationContext = initWebApplicationContext();
2.--→ initFrameworkServlet();	// 模板方法
   }
   catch (ServletException ex) {
      this.logger.error("Context initialization failed", ex);
      throw ex;
   }
   catch (RuntimeException ex) {
      this.logger.error("Context initialization failed", ex);
      throw ex;
   }

   if (this.logger.isInfoEnabled()) {
      long elapsedTime = System.currentTimeMillis() - startTime;
      this.logger.info("FrameworkServlet '" + getServletName() + "': initialization completed in " +
            elapsedTime + " ms");
   }
}
```

#### 9.3.1 initWebApplicationContext() 

该方法做了三件事：

1. 获取spring的根容器rootContext
2. 设置webApplicationContext并根据情况调用onRefresh方法
3. 将webApplicationContext设置到ServletContext中

代码如下：

```java
@Override
protected final void initServletBean() throws ServletException {
   getServletContext().log("Initializing Spring FrameworkServlet '" + getServletName() + "'");
   if (this.logger.isInfoEnabled()) {
      this.logger.info("FrameworkServlet '" + getServletName() + "': initialization started");
   }
   long startTime = System.currentTimeMillis();

   try {
      this.webApplicationContext = initWebApplicationContext();
      initFrameworkServlet();
   }
   catch (ServletException ex) {
      this.logger.error("Context initialization failed", ex);
      throw ex;
   }
   catch (RuntimeException ex) {
      this.logger.error("Context initialization failed", ex);
      throw ex;
   }

   if (this.logger.isInfoEnabled()) {
      long elapsedTime = System.currentTimeMillis() - startTime;
      this.logger.info("FrameworkServlet '" + getServletName() + "': initialization completed in " +
            elapsedTime + " ms");
   }
}
```

##### 1. 获取spring的根容器rootContext

默认情况下spring会将自己的容器设置成ServletContext的属性，默认根容器的key为：org.springframework.web.context.WebApplicationContext.ROOT，WebApplicationContext中定义了该属性：

```java
String ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE = WebApplicationContext.class.getName() + ".ROOT";
```

所以获取根容器只需要调用ServletContext的getAttribute即可

##### 2. 设置webApplicationContext并根据情况调用onRefresh方法

设置webApplicationContext有三种情况：

1. webApplicationContext已经存在；
   这种方法主要在Servlet3.0以后中，因为提供了ServletContext.addServlet方式注册Servlet，这时就可以在新建FrameworkServlet和其子类时通过构造方法传递webApplicationContext了

2. webApplicationContext已经在ServletContext中。
   例如在ServletContext中有一个叫haha的wac，那么配置contextAttribute属性即可

   ```xml
   <servlet>
       <servlet-name>let'sGo</servlet-name>
       <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
       <init-param>
           <param-name>contextAttribute</param-name>
           <param-value>haha</param-value>
       </init-param>
       <load-on-startup>1</load-on-startup>
   </servlet>
   ```

3. 在前面两种情况无效下，则创建一个（一般情况下是这个）
   具体方法是：`FrameworkServlet#createWebApplicationContext(WebApplicationContext xx)`
   内部调用了：`configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac)`
   知识点：ApplicationListener机制

##### 3. 将webApplicationContext设置到ServletContext中

最后根据publishContext标志判断是否将创建出来的wac放到ServletContext中（publishContext属性可在initParam那配置）

总结下前面提到可配置的属性：
![](http://ww1.sinaimg.cn/large/8747d788gy1fu0fhoqldnj21qg0bhjym.jpg)





