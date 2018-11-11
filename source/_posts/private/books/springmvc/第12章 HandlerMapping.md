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



















