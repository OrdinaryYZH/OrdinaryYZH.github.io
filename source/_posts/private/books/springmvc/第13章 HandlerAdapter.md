## 第13章 HandlerAdapter

HandlerAdapter的作用是使用Handler来干活，类图如下：

![](https://ws1.sinaimg.cn/large/8747d788gy1fus1fqg5gqj21yn09643h.jpg)

注意下HandlerAdapter接口的三个方法：

*  boolean supports(Object handler);
*  ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;
*  long getLastModified(HttpServletRequest request, Object handler);

HttpRequestHandlerAdapter、SimpleControllerHandlerAdapter和SimpleServletHandlerAdapter分别适配HttpRequestHandler、Servlet和Controller类型的Handler，处理方法非常简单：

例如：HttpRequestHandlerAdapter#handle：

```java
public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
      throws Exception {

   ((HttpRequestHandler) handler).handleRequest(request, response);
   return null;
}
```

### 13.1 RequestMappingHandlerAdapter概述

#### 13.1.1 AbstractHandlerMethodAdapter

该类很简单，实现了Order接口，重写了Handler的三个方法，并且实现方式都调用了模板方法xxxInternal(),例如:

```java
@Override
public final boolean supports(Object handler) {
    return (handler instanceof HandlerMethod && supportsInternal((HandlerMethod) handler));
}

protected abstract boolean supportsInternal(HandlerMethod handlerMethod);
```

#### 13.1.2 RequestMappingHandlerAdapter

RequestMappingHandlerAdapter实现的三个模板方法内容如下：

* supportsInternal(): 直接返回true
* getLastModifiedInternal(): 直接返回-1
* handleInternal(): 最重要的方法，使用Handler处理请求，具体过程大致分为三步：
  1. 备好处理器所需要的参数（这一步最麻烦）
  2. 使用处理器处理请求
  3. 处理返回值（将不同类型的返回值同意处理成ModelAndView）

搞清楚3个问题：

* 都有哪些参数需要绑定

  除了处理器的方法之外，还有注释了@ModelAttribute和@InitBinder的方法

* 参数的值的来源

  1. request中相关的参数，包括url中的参数、post过来的参数以及请求头所包含的值
  2. cookie中的参数
  3. session中的参数
  4. 设置到FlashMap中的参数，这种参数主要用于redirect的参数传递
  5. SessionAttributes传递的参数
  6. 通过注释了@ModelAttribute的方法进行设置的参数

* 具体进行绑定的方法

  参数的具体解析是使用HandlerMethodArgumentResolver类型的组件完成的。

  前面说到注释了@InitBinder的方法也需要绑定参数，所以@InitBinder注释的方法也需要ArgumentResolver来解析参数，但是他使用的和Handler使用的不是同一套ArgumentResolver。

  另外，注释了@ModelAttribute的方法也需要绑定参数，他使用的ArgumentResolver和Handler是同一套。

> 多知道点：@InitBinder @ModelAttribute @ControllerAdvice以及ResponseBodyAdvice接口的作用 P164
>
> 参考：[SpringMVC的@InitBinder注解使用](http://blog.51cto.com/simplelife/1919597)

### 13.2 RequestMappingHandlerAdapter自身结构

#### 13.2.1 创建RequestMappingHandlerAdapter之器

RequestMappingHandlerAdapter的初始化在afterPropertiesSet方法中实现，工作是初始化以下6个属性：

1. **modelAttributeAdviceCache**和**initBinderAdviceCache**：分别用于缓存@ControllerAdvice注释的类里面注释了@ModelAttribute和@InitBinder的方法，也就是全局@ModelAttribute和@InitBinder的方法。而处理器自己@ModelAttribute和@InitBinder的方法是在第一次使用处理器处理请求时缓存起来的
2. **responseBodyAdvice**：保存实现了ResponseBodyAdvice接口、可以修改返回的ResponseBody的类
3. **argumentResolvers**：给Handler和@ModelAttribute方法设置参数
4. **initBinderArgumentResolvers**：给@InitBinder方法设置参数
5. **returnValueHandlers**：给Handler的返回值处理成ModelAndView的类型

RequestMappingHandlerAdapter#afterPropertiesSet

```java
@Override
public void afterPropertiesSet() {
    // 初始化@ControllerAdvice的类相关属性
    initControllerAdviceCache();

    if (this.argumentResolvers == null) {
        List<HandlerMethodArgumentResolver> resolvers = getDefaultArgumentResolvers();
        this.argumentResolvers = new HandlerMethodArgumentResolverComposite().addResolvers(resolvers);
    }
    if (this.initBinderArgumentResolvers == null) {
        List<HandlerMethodArgumentResolver> resolvers = getDefaultInitBinderArgumentResolvers();
        this.initBinderArgumentResolvers = new HandlerMethodArgumentResolverComposite().addResolvers(resolvers);
    }
    if (this.returnValueHandlers == null) {
        List<HandlerMethodReturnValueHandler> handlers = getDefaultReturnValueHandlers();
        this.returnValueHandlers = new HandlerMethodReturnValueHandlerComposite().addHandlers(handlers);
    }
}
```

RequestMappingHandlerAdapter#initControllerAdviceCache

```java
private void initControllerAdviceCache() {
    if (getApplicationContext() == null) {
        return;
    }

    List<ControllerAdviceBean> beans = ControllerAdviceBean.findAnnotatedBeans(getApplicationContext());
    AnnotationAwareOrderComparator.sort(beans);

    List<Object> requestResponseBodyAdviceBeans = new ArrayList<Object>();

    for (ControllerAdviceBean bean : beans) {
        // 查找注释了@ModelAttribute且没注释@RequestMapping的方法
        Set<Method> attrMethods = MethodIntrospector.selectMethods(bean.getBeanType(), MODEL_ATTRIBUTE_METHODS);
        if (!attrMethods.isEmpty()) {
            this.modelAttributeAdviceCache.put(bean, attrMethods);
        }
        // 查找注释了@InitBinder的方法
        Set<Method> binderMethods = MethodIntrospector.selectMethods(bean.getBeanType(), INIT_BINDER_METHODS);
        if (!binderMethods.isEmpty()) {
            this.initBinderAdviceCache.put(bean, binderMethods);
        }
        // 查找实现了ResponseBodyAdvice接口的类
        if (RequestBodyAdvice.class.isAssignableFrom(bean.getBeanType())) {
            requestResponseBodyAdviceBeans.add(bean);
        }
        if (ResponseBodyAdvice.class.isAssignableFrom(bean.getBeanType())) {
            requestResponseBodyAdviceBeans.add(bean);
        }
    }

    if (!requestResponseBodyAdviceBeans.isEmpty()) {
        this.requestResponseBodyAdvice.addAll(0, requestResponseBodyAdviceBeans);
    }
}
```

getDefaultXXX方法，以下为RequestMappingHandlerAdapter#getDefaultArgumentResolvers：

```java
private List<HandlerMethodArgumentResolver> getDefaultArgumentResolvers() {
    List<HandlerMethodArgumentResolver> resolvers = new ArrayList<HandlerMethodArgumentResolver>();

    // Annotation-based argument resolution
    resolvers.add(new RequestParamMethodArgumentResolver(getBeanFactory(), false));
    resolvers.add(new RequestParamMapMethodArgumentResolver());
    resolvers.add(new PathVariableMethodArgumentResolver());
    resolvers.add(new PathVariableMapMethodArgumentResolver());
    resolvers.add(new MatrixVariableMethodArgumentResolver());
    resolvers.add(new MatrixVariableMapMethodArgumentResolver());
    resolvers.add(new ServletModelAttributeMethodProcessor(false));
    resolvers.add(new RequestResponseBodyMethodProcessor(getMessageConverters(), this.requestResponseBodyAdvice));
    resolvers.add(new RequestPartMethodArgumentResolver(getMessageConverters(), this.requestResponseBodyAdvice));
    resolvers.add(new RequestHeaderMethodArgumentResolver(getBeanFactory()));
    resolvers.add(new RequestHeaderMapMethodArgumentResolver());
    resolvers.add(new ServletCookieValueMethodArgumentResolver(getBeanFactory()));
    resolvers.add(new ExpressionValueMethodArgumentResolver(getBeanFactory()));
    resolvers.add(new SessionAttributeMethodArgumentResolver());
    resolvers.add(new RequestAttributeMethodArgumentResolver());

    // Type-based argument resolution
    resolvers.add(new ServletRequestMethodArgumentResolver());
    resolvers.add(new ServletResponseMethodArgumentResolver());
    resolvers.add(new HttpEntityMethodProcessor(getMessageConverters(), this.requestResponseBodyAdvice));
    resolvers.add(new RedirectAttributesMethodArgumentResolver());
    resolvers.add(new ModelMethodProcessor());
    resolvers.add(new MapMethodProcessor());
    resolvers.add(new ErrorsMethodArgumentResolver());
    resolvers.add(new SessionStatusMethodArgumentResolver());
    resolvers.add(new UriComponentsBuilderMethodArgumentResolver());

    // Custom arguments
    if (getCustomArgumentResolvers() != null) {
        resolvers.addAll(getCustomArgumentResolvers());
    }

    // Catch-all
    resolvers.add(new RequestParamMethodArgumentResolver(getBeanFactory(), true));
    resolvers.add(new ServletModelAttributeMethodProcessor(true));

    return resolvers;
}
```

#### 13.2.2 RequestMappingHandlerAdapter之用

入口方法就是handleInternal()：

```java
@Override
protected ModelAndView handleInternal(HttpServletRequest request,
        HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

    ModelAndView mav;
    checkRequest(request);

    // Execute invokeHandlerMethod in synchronized block if required.
    if (this.synchronizeOnSession) {
        HttpSession session = request.getSession(false);
        if (session != null) {
            Object mutex = WebUtils.getSessionMutex(session);
            synchronized (mutex) {
       ---→     mav = invokeHandlerMethod(request, response, handlerMethod);
            }
        }
        else {
            // No HttpSession available -> no mutex necessary
       ---→ mav = invokeHandlerMethod(request, response, handlerMethod);
        }
    }
    else {
        // No synchronization on session demanded at all...
        mav = invokeHandlerMethod(request, response, handlerMethod);
    }

   	// 给response设置缓存
    if (!response.containsHeader(HEADER_CACHE_CONTROL)) {
        if (getSessionAttributesHandler(handlerMethod).hasSessionAttributes()) {
            applyCacheSeconds(response, this.cacheSecondsForSessionAttributeHandlers);
        }
        else {
            prepareResponse(response);
        }
    }

    return mav;
}
```

书上的代码跟4.3.18有点差别，但是大致思路一样。

handleInternal()方法主要就是做以下2件事情

1. 使用HandlerMethod处理
2. 设置response缓存

书上提到了@SessionAttributes注解的用法，参考P177，但是感觉用处不大。

主要看`invokeHandlerMethod`方法:

##### invokeHandleMethod方法

先看下代码：

```java
protected ModelAndView invokeHandlerMethod(HttpServletRequest request,
        HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {

    ServletWebRequest webRequest = new ServletWebRequest(request, response);
    try {
  ---→  WebDataBinderFactory binderFactory = getDataBinderFactory(handlerMethod);
  ---→  ModelFactory modelFactory = getModelFactory(handlerMethod, binderFactory);

  ---→  ServletInvocableHandlerMethod invocableMethod = createInvocableHandlerMethod(handlerMethod);
        invocableMethod.setHandlerMethodArgumentResolvers(this.argumentResolvers);
        invocableMethod.setHandlerMethodReturnValueHandlers(this.returnValueHandlers);
        invocableMethod.setDataBinderFactory(binderFactory);
        invocableMethod.setParameterNameDiscoverer(this.parameterNameDiscoverer);

  ---→  ModelAndViewContainer mavContainer = new ModelAndViewContainer();
        mavContainer.addAllAttributes(RequestContextUtils.getInputFlashMap(request));
        modelFactory.initModel(webRequest, mavContainer, invocableMethod);
        mavContainer.setIgnoreDefaultModelOnRedirect(this.ignoreDefaultModelOnRedirect);

        AsyncWebRequest asyncWebRequest = WebAsyncUtils.createAsyncWebRequest(request, response);
        asyncWebRequest.setTimeout(this.asyncRequestTimeout);

        WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
        asyncManager.setTaskExecutor(this.taskExecutor);
        asyncManager.setAsyncWebRequest(asyncWebRequest);
        asyncManager.registerCallableInterceptors(this.callableInterceptors);
        asyncManager.registerDeferredResultInterceptors(this.deferredResultInterceptors);

        if (asyncManager.hasConcurrentResult()) {
            Object result = asyncManager.getConcurrentResult();
            mavContainer = (ModelAndViewContainer) asyncManager.getConcurrentResultContext()[0];
            asyncManager.clearConcurrentResult();
            if (logger.isDebugEnabled()) {
                logger.debug("Found concurrent result value [" + result + "]");
            }
            invocableMethod = invocableMethod.wrapConcurrentResult(result);
        }

  ---→  invocableMethod.invokeAndHandle(webRequest, mavContainer);
        if (asyncManager.isConcurrentHandlingStarted()) {
            return null;
        }

  ---→ return getModelAndView(mavContainer, modelFactory, webRequest);
    }
    finally {
        webRequest.requestCompleted();
    }
}
```

该方法的内容主要是：

1. 创建了ServletWebRequest（构造时使用了request和response）

2. 初始化`WebDataBinderFactory`

   作用：从名字就能看出来这是用来创建`WebDataBinder`的，`WebDataBinder`用于参数绑定，实现参数跟String之间的类型转换。ArgumentResolver在进行参数解析的过程中使用到`WebDataBinder`，ModelFactory在更新Model时也会用到它。

   创建的过程就是找出@InitBinder的方法，并将它们新建出`ServletRequestDataBinderFactory`类型的`WebDataBinderFactory`

3. 初始化`ModelFactory`

   `ModelFactory`是用来处理Model的，主要是这两个功能：

   * 在处理器具体处理之前对Model进行初始化
   * 在处理完请求后对Model参数进行更新

   getModelFactory方法主要是：

   1. 获取SessionAttributesHandler
   2. 从处理器中获取@ModelAttribute且没有@RequestMapping的method
   3. 从@ControllerAdvice类里找出@ModelAttribute的method
   4. 最后使用注释了@ModelAttribute的方法、WebDataBinderFactory和SessionAttributesHandler创建`ModelFactory`

4. 初始化`ServletInvocableHandlerMethod`

（忽略异步处理）

5. 新建传递参的`ModelAndViewContainer`容器，并将相应参数设置到其Model中

6. 执行请求

7. 请求处理完后的后置处理
