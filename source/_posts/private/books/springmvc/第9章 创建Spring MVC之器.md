## 第9章 创建Spring MVC之器

### 9.1 整体结构介绍

类图如下：

![](http://ww1.sinaimg.cn/large/8747d788gy1fu24td3cs0j21kw0vnnam.jpg)

知识点：

1. **XXXAware的作用**(Spring提供XXX给Bean)：Spring提供XXX给Bean：如果在某个类里面想要使用spring的一些东西，实现该接口后，spring就会把想要的东西注入，接收的方式是实现set-XXX方法 ；

   例如：ApplicationContextAwareProcessor实现该项工作，AbstractAutowireCapableBeanFactory#applyBeanPostProcessorsBeforeInitialization 方法会遍历PostProcessor。

   > ApplicationContextAwareProcessor的实现：
   >
   > ![](https://ws1.sinaimg.cn/large/8747d788gy1fx2b18uv4tj21jr1cutiq.jpg)

   BeanNameAware、BeanClassLoaderAware和BeanFactoryAware不是在这个Processor中设置的，是在BeanProcessor调用之前设置的：详情看：

   AbstractAutowireCapableBeanFactory#initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd)

2. **EnvironmentCapable的作用**？(Bean提供Environment给Spring)  实现该接口的类代表其能够提供Environment，方法为：getEnvironment()

3. **Environment都有啥**？(HttpServletBean中Environment使用的是Standard-Servlet-Environment)
   * ServletContext **ServletContextPropertySource** 
   * ServletConfig   **ServletConfigPropertySource** 
   * JndiProperty     **JndiPropertySource** 
   * 系统属性           **MapPropertySource** 
   * 系统环境变量    **SystemEnvironmentPropertySource** 

### 9.2 HttpServletBean

覆盖了Servlet的`init()`方法，如下：

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
> 首先将Servlet中配置的参数使用BeanWrapper设置到~~DispatcherServlet~~自身的相关属性，然后调用模板方法initServletBean()，子类就通过这个方法初始化 

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
protected WebApplicationContext initWebApplicationContext() {
    WebApplicationContext rootContext =
            WebApplicationContextUtils.getWebApplicationContext(getServletContext());
    WebApplicationContext wac = null;

    if (this.webApplicationContext != null) {
        // A context instance was injected at construction time -> use it
        wac = this.webApplicationContext;
        if (wac instanceof ConfigurableWebApplicationContext) {
            ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;
            if (!cwac.isActive()) {
                // The context has not yet been refreshed -> provide services such as
                // setting the parent context, setting the application context id, etc
                if (cwac.getParent() == null) {
                    // The context instance was injected without an explicit parent -> set
                    // the root application context (if any; may be null) as the parent
                    cwac.setParent(rootContext);
                }
                configureAndRefreshWebApplicationContext(cwac);
            }
        }
    }
    if (wac == null) {
        // No context instance was injected at construction time -> see if one
        // has been registered in the servlet context. If one exists, it is assumed
        // that the parent context (if any) has already been set and that the
        // user has performed any initialization such as setting the context id
        wac = findWebApplicationContext();
    }
    if (wac == null) {
        // No context instance is defined for this servlet -> create a local one
        wac = createWebApplicationContext(rootContext);
    }

    if (!this.refreshEventReceived) {
        // Either the context is not a ConfigurableApplicationContext with refresh
        // support or the context injected at construction time had already been
        // refreshed -> trigger initial onRefresh manually here.
        onRefresh(wac);
    }

    if (this.publishContext) {
        // Publish the context as a servlet context attribute.
        String attrName = getServletContextAttributeName();
        getServletContext().setAttribute(attrName, wac);
        if (this.logger.isDebugEnabled()) {
            this.logger.debug("Published WebApplicationContext of servlet '" + getServletName() +
                    "' as ServletContext attribute with name [" + attrName + "]");
        }
    }

    return wac;
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



###### 知识点：ApplicationListener机制

   createWebApplicationContext(ApplicationContext parent)：

   ```java
   protected WebApplicationContext createWebApplicationContext(ApplicationContext parent) {
       Class<?> contextClass = getContextClass();
       if (this.logger.isDebugEnabled()) {
           this.logger.debug("Servlet with name '" + getServletName() +
                   "' will try to create custom WebApplicationContext context of class '" +
                   contextClass.getName() + "'" + ", using parent context [" + parent + "]");
       }
       if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
           throw new ApplicationContextException(
                   "Fatal initialization error in servlet with name '" + getServletName() +
                   "': custom WebApplicationContext class [" + contextClass.getName() +
                   "] is not of type ConfigurableWebApplicationContext");
       }
       ConfigurableWebApplicationContext wac =
               (ConfigurableWebApplicationContext) BeanUtils.instantiateClass(contextClass);
   
       wac.setEnvironment(getEnvironment());
       wac.setParent(parent);
       wac.setConfigLocation(getContextConfigLocation());
   
       configureAndRefreshWebApplicationContext(wac);
   
       return wac;
   }
   ```

   configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac)：

   ```java
   protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac) {
       if (ObjectUtils.identityToString(wac).equals(wac.getId())) {
           // The application context id is still set to its original default value
           // -> assign a more useful id based on available information
           if (this.contextId != null) {
               wac.setId(this.contextId);
           }
           else {
               // Generate default id...
               wac.setId(ConfigurableWebApplicationContext.APPLICATION_CONTEXT_ID_PREFIX +
                       ObjectUtils.getDisplayString(getServletContext().getContextPath()) + '/' + getServletName());
           }
       }
   
       wac.setServletContext(getServletContext());
       wac.setServletConfig(getServletConfig());
       wac.setNamespace(getNamespace());
       wac.addApplicationListener(new SourceFilteringListener(wac, new ContextRefreshListener()));	// 添加listener(一个内部类实现了ApplicationListener接口)
   
       // The wac environment's #initPropertySources will be called in any case when the context
       // is refreshed; do it eagerly here to ensure servlet property sources are in place for
       // use in any post-processing or initialization that occurs below prior to #refresh
       ConfigurableEnvironment env = wac.getEnvironment();
       if (env instanceof ConfigurableWebEnvironment) {
           ((ConfigurableWebEnvironment) env).initPropertySources(getServletContext(), getServletConfig());
       }
   
       postProcessWebApplicationContext(wac);
       applyInitializers(wac);
       wac.refresh();
   }
   ```


##### 3. 将webApplicationContext设置到ServletContext中

最后根据publishContext标志判断是否将创建出来的wac放到ServletContext中（publishContext属性可在initParam那配置）

总结下前面提到可配置的属性：
![](http://ww1.sinaimg.cn/large/8747d788gy1fu0fhoqldnj21qg0bhjym.jpg)



### 9.4 DispatcherServlet

onRefresh()是DispatcherServlet的入口方法。因为在FrameworkServlet中注册了Listener，所以在wac刷新后发布刷新事件时，该方法被调用。

FrameworkServlet中：

```java
private class ContextRefreshListener implements ApplicationListener<ContextRefreshedEvent> {

   @Override
   public void onApplicationEvent(ContextRefreshedEvent event) {
      FrameworkServlet.this.onApplicationEvent(event);
   }
}
```

```java
public void onApplicationEvent(ContextRefreshedEvent event) {
   this.refreshEventReceived = true;
   onRefresh(event.getApplicationContext());
}
```

---

DispatcherServlet：

```java
@Override
protected void onRefresh(ApplicationContext context) {
   initStrategies(context);
}
```

```java
protected void initStrategies(ApplicationContext context) {
   initMultipartResolver(context);
   initLocaleResolver(context);
   initThemeResolver(context);
   initHandlerMappings(context);
   initHandlerAdapters(context);
   initHandlerExceptionResolvers(context);
   initRequestToViewNameTranslator(context);
   initViewResolvers(context);
   initFlashMapManager(context);
}
```

initStrategies中调用了9个初始化方法。

下面以LocalResolver为例分析初始化方法：

```java
private void initLocaleResolver(ApplicationContext context) {
   try {
      this.localeResolver = context.getBean(LOCALE_RESOLVER_BEAN_NAME, LocaleResolver.class);
      if (logger.isDebugEnabled()) {
         logger.debug("Using LocaleResolver [" + this.localeResolver + "]");
      }
   }
   catch (NoSuchBeanDefinitionException ex) {
      // We need to use the default.
      this.localeResolver = getDefaultStrategy(context, LocaleResolver.class);
      if (logger.isDebugEnabled()) {
         logger.debug("Unable to locate LocaleResolver with name '" + LOCALE_RESOLVER_BEAN_NAME +
               "': using default [" + this.localeResolver + "]");
      }
   }
}
```

在wac中找不到时，则使用默认组件

默认的组件配置在DispatcherServlet所在的包DispatcherServlet.properties文件下配置了：

```properties
# Default implementation classes for DispatcherServlet's strategy interfaces.
# Used as fallback when no matching beans are found in the DispatcherServlet context.
# Not meant to be customized by application developers.

org.springframework.web.servlet.LocaleResolver=org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver

org.springframework.web.servlet.ThemeResolver=org.springframework.web.servlet.theme.FixedThemeResolver

org.springframework.web.servlet.HandlerMapping=org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping,\
   org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping

org.springframework.web.servlet.HandlerAdapter=org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter,\
   org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter,\
   org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter

org.springframework.web.servlet.HandlerExceptionResolver=org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerExceptionResolver,\
   org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolver,\
   org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver

org.springframework.web.servlet.RequestToViewNameTranslator=org.springframework.web.servlet.view.DefaultRequestToViewNameTranslator

org.springframework.web.servlet.ViewResolver=org.springframework.web.servlet.view.InternalResourceViewResolver

org.springframework.web.servlet.FlashMapManager=org.springframework.web.servlet.support.SessionFlashMapManager
```

这里定义了8个组件，少了处理上传的组件。（**所以如果使用上传功能的话需要引入相关的组件**）

多知道点：在spring的xml文件中通过命名空间配置的标签时怎么解析的？（略）



### 9.5 小结

主要分析了Spring MVC自身的创建过程，Spring MVC中的Servlet一共有三个层次，分别是HttpServletBean、FrameworkServlet和DispatcherServlet。

* HttpServletBean直接继承自HttpServlet，它将Servlet中的配置参数设置到相应的属性
* FrameworkServlet初始化了WebApplicationContext
* DispatcherServlet初始化了自身的9个组件















