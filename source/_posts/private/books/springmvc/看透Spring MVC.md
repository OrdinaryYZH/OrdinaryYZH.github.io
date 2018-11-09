## 第8章 SPring MVC之初体验

搭建一个最简单的Spring MVC环境：

1. maven依赖servlet-api和spring-webmvc
2. web.xml中配置DispatcherServlet
3. 创建Spring MVC xml配置文件:
   * `<mvc:annotation-driven/>`

   * `<context: component-scan base-package="xxx">`，一般还配置只扫面@Controller注解

     ```xml
     <context:component-scan base-package="com.xxx"
                             use-default-filters="false">
         <context:include-filter type="annotation"
                                 expression="org.springframework.stereotype.Controller"/>
     </context:component-scan>
     ```

4. 创建Controller（和View）

## 第9章 创建Spring MVC之器

### 9.1 整体结构介绍

类图如下：

![](http://ww1.sinaimg.cn/large/8747d788gy1fu24td3cs0j21kw0vnnam.jpg)

知识点：

1. **XXXAware的作用**(Spring提供XXX给Bean)：如果在某个类里面想要使用spring的一些东西，实现该接口后，spring就会把想要的东西注入，接收的方式是实现set-XXX方法 ；ApplicationContextAwareProcessor实现该项工作，`AbstractAutowireCapableBeanFactory#applyBeanPostProcessorsBeforeInitialization` 方法会遍历PostProcessor。
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
> 首先将Servlet中配置的参数使用BeanWrapper设置到DispatcherServlet的相关属性，然后调用模板方法initServletBean()，子类就通过这个方法初始化 

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







## 第12章 HandlerMapping

HandlerMapping类图如下：

![](http://ww1.sinaimg.cn/large/8747d788gy1fuapyod0bnj21kw0hygwq.jpg)

### 12.1 AbstractHandlerMapping

#### 12.1.1 创建AbstractHandlerMapping之器

AbstractHandlerMapping的初始化工作（在被实例化之后）方法为：`initApplicationContext()`

该方法主要是处理拦截器相关工作。

```java
protected void initApplicationContext() throws BeansException {
    extendInterceptors(this.interceptors);	// 模板方法，但没实现
    detectMappedInterceptors(this.adaptedInterceptors);	//加载容器中的拦截器到adaptedInterceptors属性中
    initInterceptors();    // 将interceptors属性中合适的拦截器添加到adaptedInterceptors属性中
}
```

> 因为`AbstractHandlerMapping`的父类`WebApplicationObjectSUpport`实现了`ApplicatonContextAware`接口，`setApplicationContext`方法调用了`initApplicationContext()`：

类图如下：

![](http://ww1.sinaimg.cn/large/8747d788gy1fuaq4ic1maj21kw0sfgwl.jpg)

附录：

1. 拦截器列表 interceptors

   用于配置SpringMVC的拦截器，配置方式有两种：

- 1. 注册HandlerMapping时通过属性设置

- 2. 通过子类的extendInterceptors钩子方法进行设置（extendInterceptors方法是在initApplicationContext中调用的）

  interceptors并不会直接使用，而是通过initInterceptors方法按照类型分配到~~mappedInterceptors~~和adaptedInterceptors中进行使用，interceptors只用于配置。

  ```java
  private final List<Object> interceptors = new ArrayList<Object>();
  ```

2. adaptedInterceptors

   被分配到adaptedInterceptors中的类型的拦截器不需要进行匹配，在getHandler中会全部添加到返回值HandlerExecutionChain里面。他 只能从 interceptors中获取。

   ```java
   private final List<HandlerInterceptor> adaptedInterceptors = new ArrayList<HandlerInterceptor>();
   ```



#### 12.1.2 AbstractHandlerMapping之用 

>  方法很简单，看下源码即可(版本是4.3.18)：

入口方法是getHandler():

```java
@Override
public final HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    Object handler = getHandlerInternal(request);	// 模板方法
    if (handler == null) {
        handler = getDefaultHandler();
    }
    if (handler == null) {
        return null;
    }
    // Bean name or resolved handler?
    if (handler instanceof String) {
        String handlerName = (String) handler;
        handler = getApplicationContext().getBean(handlerName);
    }

    HandlerExecutionChain executionChain = getHandlerExecutionChain(handler, request);
    if (CorsUtils.isCorsRequest(request)) {
        CorsConfiguration globalConfig = this.globalCorsConfigSource.getCorsConfiguration(request);
        CorsConfiguration handlerConfig = getCorsConfiguration(handler, request);
        CorsConfiguration config = (globalConfig != null ? globalConfig.combine(handlerConfig) : handlerConfig);
        executionChain = getCorsHandlerExecutionChain(request, executionChain, config);
    }
    return executionChain;
}
```

---

getHandlerExecutionChain(Object handler, HttpServletRequest request)：

```java
protected HandlerExecutionChain getHandlerExecutionChain(Object handler, HttpServletRequest request) {
    HandlerExecutionChain chain = (handler instanceof HandlerExecutionChain ?
            (HandlerExecutionChain) handler : new HandlerExecutionChain(handler));

    String lookupPath = this.urlPathHelper.getLookupPathForRequest(request);
    for (HandlerInterceptor interceptor : this.adaptedInterceptors) {
        if (interceptor instanceof MappedInterceptor) {
            MappedInterceptor mappedInterceptor = (MappedInterceptor) interceptor;
            if (mappedInterceptor.matches(lookupPath, this.pathMatcher)) {
                chain.addInterceptor(mappedInterceptor.getInterceptor());
            }
        }
        else {
            chain.addInterceptor(interceptor);
        }
    }
    return chain;
}
```

### 12.2 AbstractUrlHandlerMapping系列（略）

#### 12.2.1 AbstractUrlHandlerMapping 

#### 12.2.2 SimpleUrlHandlerMapping

#### 12.2.3 AbstractDetectingUrlHandlerMapping

 

 ### 12.3 AbstractHandlerMethodMapping系列

> AbstractHandlerMethodMapping系列将Method作为Handler来使用，他专门有一个类型 -- HandlerMethod

#### 12.3.1 创建AbstractHandlerMethodMapping系列之器

> 先一句话总结：该类实例化后，初始化中查找HandlerMethod，并将结果放到内部类属性`MappingRegistry`中：
>
> ![](http://ww1.sinaimg.cn/large/8747d788gy1fublgw06j9j20sg0d0dqk.jpg)

入口方法为`afterPropertiesSet()`，因为该类实现了`InitilalizingBean`接口.

```java
@Override
public void afterPropertiesSet() {
    initHandlerMethods();
}
```

AbstractHandlerMethodMapping#initHandlerMethods

```java
/**
    * Scan beans in the ApplicationContext, detect and register handler methods.
    * @see #isHandler(Class)
    * @see #getMappingForMethod(Method, Class)
    * @see #handlerMethodsInitialized(Map)
    */
protected void initHandlerMethods() {
    if (logger.isDebugEnabled()) {
        logger.debug("Looking for request mappings in application context: " + getApplicationContext());
    }
    String[] beanNames = (this.detectHandlerMethodsInAncestorContexts ?
            BeanFactoryUtils.beanNamesForTypeIncludingAncestors(getApplicationContext(), Object.class) :
            getApplicationContext().getBeanNamesForType(Object.class));

    for (String beanName : beanNames) {
        if (!beanName.startsWith(SCOPED_TARGET_NAME_PREFIX)) {
            Class<?> beanType = null;
            try {
                beanType = getApplicationContext().getType(beanName);
            }
            catch (Throwable ex) {
                // An unresolvable bean type, probably from a lazy bean - let's ignore it.
                if (logger.isDebugEnabled()) {
                    logger.debug("Could not resolve target class for bean with name '" + beanName + "'", ex);
                }
            }
    ----→   if (beanType != null && isHandler(beanType)) {	// 找到handler，RequestMappingHandlerMapping有实现：Controller || RequestMapping注解
    -----→      detectHandlerMethods(beanName);	//handler中找method
            }
        }
    }
    handlerMethodsInitialized(getHandlerMethods());	// 模板方法，但是没有类实现
}
```

==AbstractHandlerMethodMapping#detectHandlerMethods==

```java
/**
    * Look for handler methods in a handler.
    * @param handler the bean name of a handler or a handler instance
    */
protected void detectHandlerMethods(final Object handler) {
    Class<?> handlerType = (handler instanceof String ?
            getApplicationContext().getType((String) handler) : handler.getClass());
    final Class<?> userType = ClassUtils.getUserClass(handlerType);

    Map<Method, T> methods = MethodIntrospector.selectMethods(userType,
            new MethodIntrospector.MetadataLookup<T>() {
                @Override
                public T inspect(Method method) {
                    try {
        -----------→    return getMappingForMethod(method, userType);
                    }
                    catch (Throwable ex) {
                        throw new IllegalStateException("Invalid mapping on handler class [" +
                                userType.getName() + "]: " + method, ex);
                    }
                }
            });

    if (logger.isDebugEnabled()) {
        logger.debug(methods.size() + " request handler methods found on " + userType + ": " + methods);
    }
    for (Map.Entry<Method, T> entry : methods.entrySet()) {
        Method invocableMethod = AopUtils.selectInvocableMethod(entry.getKey(), userType);
        T mapping = entry.getValue();
    --→ registerHandlerMethod(handler, invocableMethod, mapping);
    }
}
```

提到的知识点：

1. HandlerMethod类
2. RequestMappingInfo和RequestCondition接口
   ![](http://ww1.sinaimg.cn/large/8747d788gy1fubm3ohmerj21kw0f148u.jpg)

#### 12.3.2 AbstractHandlerMethodMapping系列之用

通过`getHandlerInternal`方法被调用：

```java
/**
    * Look up a handler method for the given request.
    */
@Override
protected HandlerMethod getHandlerInternal(HttpServletRequest request) throws Exception {
    String lookupPath = getUrlPathHelper().getLookupPathForRequest(request);
    if (logger.isDebugEnabled()) {
        logger.debug("Looking up handler method for path " + lookupPath);
    }
    this.mappingRegistry.acquireReadLock();
    try {
 ---→   HandlerMethod handlerMethod = lookupHandlerMethod(lookupPath, request);
        if (logger.isDebugEnabled()) {
            if (handlerMethod != null) {
                logger.debug("Returning handler method [" + handlerMethod + "]");
            }
            else {
                logger.debug("Did not find handler method for [" + lookupPath + "]");
            }
        }
        return (handlerMethod != null ? handlerMethod.createWithResolvedBean() : null);
    }
    finally {
        this.mappingRegistry.releaseReadLock();
    }
}
```

```java
/**
    * Look up the best-matching handler method for the current request.
    * If multiple matches are found, the best match is selected.
    * @param lookupPath mapping lookup path within the current servlet mapping
    * @param request the current request
    * @return the best-matching handler method, or {@code null} if no match
    * @see #handleMatch(Object, String, HttpServletRequest)
    * @see #handleNoMatch(Set, String, HttpServletRequest)
    */
protected HandlerMethod lookupHandlerMethod(String lookupPath, HttpServletRequest request) throws Exception {
    List<Match> matches = new ArrayList<Match>();
    List<T> directPathMatches = this.mappingRegistry.getMappingsByUrl(lookupPath);
    if (directPathMatches != null) {
        addMatchingMappings(directPathMatches, matches, request);
    }
    if (matches.isEmpty()) {
        // No choice but to go through all mappings...
        addMatchingMappings(this.mappingRegistry.getMappings().keySet(), matches, request);
    }

    if (!matches.isEmpty()) {
        Comparator<Match> comparator = new MatchComparator(getMappingComparator(request));
        Collections.sort(matches, comparator);
        if (logger.isTraceEnabled()) {
            logger.trace("Found " + matches.size() + " matching mapping(s) for [" +
                    lookupPath + "] : " + matches);
        }
        Match bestMatch = matches.get(0);
        if (matches.size() > 1) {
            if (CorsUtils.isPreFlightRequest(request)) {
                return PREFLIGHT_AMBIGUOUS_MATCH;
            }
            Match secondBestMatch = matches.get(1);
            if (comparator.compare(bestMatch, secondBestMatch) == 0) {
                Method m1 = bestMatch.handlerMethod.getMethod();
                Method m2 = secondBestMatch.handlerMethod.getMethod();
                throw new IllegalStateException("Ambiguous handler methods mapped for HTTP path '" +
                        request.getRequestURL() + "': {" + m1 + ", " + m2 + "}");
            }
        }
        handleMatch(bestMatch.mapping, lookupPath, request);
        return bestMatch.handlerMethod;
    }
    else {
        return handleNoMatch(this.mappingRegistry.getMappings().keySet(), lookupPath, request);
    }
}
```



















