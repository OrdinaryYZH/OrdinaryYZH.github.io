## 第10章 Spring MVC之用

> 本章分析Spring MVC是怎么处理请求的。

### 10.1 HttpServletBean

HttpServletBean主要参与了创建工作，并没有涉及请求的处理

### 10.2 FrameworkServlet

> FrameoworkServlet重写了service、doGet、doPost、doPut、doDelete、doOptions、doTrace方法(除了doHead..)

#### 10.2.1 service()

FrameworkServlet中重写了service()方法，添加了对Patch请求的处理，这就是请求的入口：

```java
@Override
protected void service(HttpServletRequest request, HttpServletResponse response)
      throws ServletException, IOException {

   HttpMethod httpMethod = HttpMethod.resolve(request.getMethod());
   if (httpMethod == HttpMethod.PATCH || httpMethod == null) {
      processRequest(request, response);
   }
   else {
      super.service(request, response);
   }
}
```

这里对PATCH请求做了处理，其他请求按照父类处理

> 注：HttpServlet中service()对一些7种请求路由到不同的方法处理
>
> 1. doGet() 
> 2. doHead()，调用了doGet()，返回只有header没有body的response
> 3. doPost()
> 4. doPut()
> 5. doDelete()
> 6. doOptions()
> 7. doTrace()

FrameworkServlet重写了以上除了doHead()的其他方法：

doGet()、doPost、doPut、doDelete都是调用processRequest方法；

doOptions和doTrace方法可以通过设置dispatchOptionsRequest和dispatchTraceRequest参数决定是自己处理还是父类处理（默认是false，交给父类处理，doOptions会在父类的处理结果中增加PATCH类型）。

doGet()方法如下：

```java
@Override
protected final void doGet(HttpServletRequest request, HttpServletResponse response)
      throws ServletException, IOException {

   processRequest(request, response);
}
```

doOptions()方法如下：

```java
@Override
protected void doOptions(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {

    if (this.dispatchOptionsRequest || CorsUtils.isPreFlightRequest(request)) {
        processRequest(request, response);
        if (response.containsHeader("Allow")) {
            // Proper OPTIONS response coming from a handler - we're done.
            return;
        }
    }

    // Use response wrapper for Servlet 2.5 compatibility where
    // the getHeader() method does not exist
    super.doOptions(request, new HttpServletResponseWrapper(response) {
        @Override
        public void setHeader(String name, String value) {
            if ("Allow".equals(name)) {
                value = (StringUtils.hasLength(value) ? value + ", " : "") + HttpMethod.PATCH.name();
            }
            super.setHeader(name, value);
        }
    });
}
```

##### 图解:

![](https://ws1.sinaimg.cn/large/8747d788gy1fvrnra1stxj221d14gaf1.jpg)

#### 10.2.2 processRequest()

processRequest()的核心方法是doService()，这是个模板方法，在DispatcherServlet中实现；

在doService()前后做了一些事情：

1. 对LocaleContext和RequestAttributes的设置（知识点：ThreadLocal）
2. 异步处理管理器注册拦截器
3. doService()
4. 对LocaleContext和RequestAttributes的恢复
5. 处理完后发布了ServletRequestHandledEvent消息

知识点：

1. LocaleContext：可以获取Locale
2. RequestAttributes：用于管理request和session的属性

代码如下：

```java
protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {

    long startTime = System.currentTimeMillis();
    Throwable failureCause = null;

    LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
    LocaleContext localeContext = buildLocaleContext(request);

    RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
    ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);

    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
    asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());

    initContextHolders(request, localeContext, requestAttributes);

    try {
 --→    doService(request, response);
    }
    catch (ServletException ex) {
        failureCause = ex;
        throw ex;
    }
    catch (IOException ex) {
        failureCause = ex;
        throw ex;
    }
    catch (Throwable ex) {
        failureCause = ex;
        throw new NestedServletException("Request processing failed", ex);
    }

    finally {
        resetContextHolders(request, previousLocaleContext, previousAttributes);
        if (requestAttributes != null) {
            requestAttributes.requestCompleted();
        }

        if (logger.isDebugEnabled()) {
            if (failureCause != null) {
                this.logger.debug("Could not complete request", failureCause);
            }
            else {
                if (asyncManager.isConcurrentHandlingStarted()) {
                    logger.debug("Leaving response open for concurrent processing");
                }
                else {
                    this.logger.debug("Successfully completed request");
                }
            }
        }

        publishRequestHandledEvent(request, response, startTime, failureCause);
    }
}
```



### 10.3 DispatcherServlet

DispatcherServlet中的入口方法是doService()，但是核心处理是doDispatch()方法，doService()中的处理是：

1. 判断是否include请求，是则对request的Attribute做快照备份，等doDispatch()处理完之后（如果不是异步调用且未完成？）进行还原
2. 对request设置一些属性，前面4个在之后介绍的handler和view中需要使用，后面3个都和flashMap有关，主要用于Redirect转法时参数的传递
   1. webApplicationContext
   2. localeResolver
   3. themeResolver
   4. themeSource
   5. inputFlashMap：用于保存上次请求中转发过来的属性
   6. outputFlashMap：用于保存本次请求需要转发的属性
   7. flashMapManager：用于管理上述2个属性
3. 调用doDispatch(request, response)

代码如下：

```java
@Override
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
    if (logger.isDebugEnabled()) {
        String resumed = WebAsyncUtils.getAsyncManager(request).hasConcurrentResult() ? " resumed" : "";
        logger.debug("DispatcherServlet with name '" + getServletName() + "'" + resumed +
                " processing " + request.getMethod() + " request for [" + getRequestUri(request) + "]");
    }

    // Keep a snapshot of the request attributes in case of an include,
    // to be able to restore the original attributes after the include.
    Map<String, Object> attributesSnapshot = null;
    if (WebUtils.isIncludeRequest(request)) {
        attributesSnapshot = new HashMap<String, Object>();
        Enumeration<?> attrNames = request.getAttributeNames();
        while (attrNames.hasMoreElements()) {
            String attrName = (String) attrNames.nextElement();
            if (this.cleanupAfterInclude || attrName.startsWith(DEFAULT_STRATEGIES_PREFIX)) {
                attributesSnapshot.put(attrName, request.getAttribute(attrName));
            }
        }
    }

    // Make framework objects available to handlers and view objects.
    request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
    request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
    request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
    request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());

    FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
    if (inputFlashMap != null) {
        request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
    }
    request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
    request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);

    try {
-->     doDispatch(request, response);
    }
    finally {
        if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
            // Restore the original attribute snapshot, in case of an include.
            if (attributesSnapshot != null) {
                restoreAttributesAfterInclude(request, attributesSnapshot);
            }
        }
    }
}
```

#### 10.3.1 重定向传递参数的用法

我们只需要在redirect之前将需要传递的参数写入outputFlashMap属性即可

举几个例子：

1. 使用RequestContextHolder，写法太复杂..

   ```java
   ((FlashMap) ((ServletRequestAttributes) (RequestContextHolder.getRequestAttributes())).getRequest().getAttribute(DispatcherServlet.OUTPUT_FLASH_MAP_ATTRIBUTE)).put("name", "张三");
   ```

2. 通过传入的RedirectAttributes参数设置

   1. addFlashAttribute：保存到outputFlashMap中，跟上述方法效果一样
   2. addAttribute：参数会拼接到url上

3. 使用RequestContextUtils
   `RequestContextUtils.getOutputFlashMap(HttpServletRequest request)`

#### 10.3.2 doDispatch

doDispatch()方法做了以下事情：

1. 根据request找到Handler
2. 根据Handler找到对应的HandlerAdapter
3. HandlerAdapter使用Handler处理请求
4. 调用processDispatchResult()方法处理上面处理之后的结果（包含找到View并渲染输出给用户）

代码如下：

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    boolean multipartRequestParsed = false;

    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

    try {
        ModelAndView mv = null;
        Exception dispatchException = null;

        try {
            processedRequest = checkMultipart(request);
            multipartRequestParsed = (processedRequest != request);

            // Determine handler for the current request.
        --→ mappedHandler = getHandler(processedRequest);
            if (mappedHandler == null || mappedHandler.getHandler() == null) {
                noHandlerFound(processedRequest, response);
                return;
            }

            // Determine handler adapter for the current request.
        --→ HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

            // Process last-modified header, if supported by the handler.
            String method = request.getMethod();
            boolean isGet = "GET".equals(method);
            if (isGet || "HEAD".equals(method)) {
                long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                if (logger.isDebugEnabled()) {
                    logger.debug("Last-Modified value for [" + getRequestUri(request) + "] is: " + lastModified);
                }
                if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
                    return;
                }
            }

            if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                return;
            }

            // Actually invoke the handler.
        --→ mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

            if (asyncManager.isConcurrentHandlingStarted()) {
                return;
            }

            applyDefaultViewName(processedRequest, mv);
            mappedHandler.applyPostHandle(processedRequest, response, mv);
        }
        catch (Exception ex) {
            dispatchException = ex;
        }
        catch (Throwable err) {
            // As of 4.3, we're processing Errors thrown from handler methods as well,
            // making them available for @ExceptionHandler methods and other scenarios.
            dispatchException = new NestedServletException("Handler dispatch failed", err);
        }
    --→ processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    }
    catch (Exception ex) {
        triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
    }
    catch (Throwable err) {
        triggerAfterCompletion(processedRequest, response, mappedHandler,
                new NestedServletException("Handler processing failed", err));
    }
    finally {
        if (asyncManager.isConcurrentHandlingStarted()) {
            // Instead of postHandle and afterCompletion
            if (mappedHandler != null) {
                mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
            }
        }
        else {
            // Clean up any resources used by a multipart request.
            if (multipartRequestParsed) {
                cleanupMultipart(processedRequest);
            }
        }
    }
}
```



### 10.4 doDispatcher结构

流程如下图：

![](https://ws1.sinaimg.cn/large/8747d788gy1fvpq598qigj22ve3h7has.jpg)

### 10.5 小结

本章整体分析了Spring MVC中请求处理的过程。首先对三个Servlet进行了分析，然后单独分析了DispatcherServlet中的doDispat方法。

三个 Servlet的处理过程大致功能如下:

- HttpservletBean：没有参与实际请求的处理。
- Framework Servlet：将不同类型的请求合并到了 processRequest方法统一处理,
  processRequest方法中做了三件事:
  - 调用了 doService模板方法具体处理请求。
  - 将当前请求的 Locale Context和 ServletrequestAttributes在处理请求前设置到了Locale contextholder和 Request ContextHolder,并在请求处理完成后恢复,
  - 请求处理完后发布了 ServletRequestHandledEvent消息。
- Dispatcher Servlet：doService方法给 request设置了一些属性并将请求交给 dispatch方法具体处理。

Dispatcher Servlet中的 dispatch方法完成 Spring MVC中请求处理过程的顶层设计,它使用 Dispatcher Servlet中的九大组件完成了具体的请求处理。另外 HandlerMapping、 Handler和 HandlerAdapter这三个概念的含义以及它们之间的关系也非常重要。