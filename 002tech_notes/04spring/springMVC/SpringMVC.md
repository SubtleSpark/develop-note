# SpringMVC
## 1 请求处理源码分析（请求映射原理）
### 1.1 RequestMappingHandlerMapping 属性
![[attachments/Pasted image 20221020233913.png]]
### 1.2 请求映射大概流程
![[attachments/Pasted image 20221015234637.png  | 500]]
FrameworkServlet 实现了所有的 doGet、doPost、doDelete 等方法。这些方法最终都会调用 `processRequest()` 来处理请求。
`processRequest()` 调用了 `doService()`，`doService()`调用了`doDispatch()`。该方法是请求映射的核心。

1. `DispatcherServlet`接收请求 Request。
2. 调用`doService()`方法。这个方法是由 Servlet 接口定义。所有Servlet子类必须实现。
3. 在`doService()`方法中，调用了`doDispatch()`。该方法是请求映射的核心。
4. `DispatcherServlet`中的`handlerMappings`属性保存了所有的HandlerMapping。遍历所有的 HandlerMapping 获取可以处理该请求的 Handler。
	1. 其中有一个RequestMappingHandlerMapping。RequestMappingHandlerMapping会根据@RequestMapping 注解标注的内容，将请求url -> 具体执行方法的映射保存在一个 Map 中。
![[002tech_notes/04spring/springMVC/请求调用链路.svg]]


## 2 参数处理源码
1.  HandlerMapping中找到能处理请求的Handler（Controller.method()）
2. 为当前Handler 找一个适配器 HandlerAdapter； **RequestMappingHandlerAdapter**
3. 适配器执行目标方法并确定方法参数的每一个值

### 2.1 HandlerAdapter
如下图默认有4种适配器，常用的是前两个
- 支持方法上标注 @RequestMapping 注解的适配器
- 支持函数式编程的适配器

![[attachments/Pasted image 20221020235602.png]]

### 2.2 执行目标方法



### 2.3 参数解析器
![[attachments/Pasted image 20221021002618.png]]

### 2.4 返回值处理器
![[attachments/Pasted image 20221021004342.png | 800]]

## 3 响应处理





## 4 doDispatch 流程注释
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
         // 获取可以处理该请求的 handler 
         mappedHandler = getHandler(processedRequest);  
         if (mappedHandler == null) {  
            noHandlerFound(processedRequest, response);  
            return;         }  
  
         // Determine handler adapter for the current request.  
         // 针对该请求获取一个适配器 HandlerAdapter
         HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());  
  
         // Process last-modified header, if supported by the handler.  
         String method = request.getMethod();  
         boolean isGet = HttpMethod.GET.matches(method);  
         if (isGet || HttpMethod.HEAD.matches(method)) {  
            long lastModified = ha.getLastModified(request, mappedHandler.getHandler());  
            if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {  
               return;  
            }  
         }  
         /* 
         正序调用拦截器的preHandle方法，
         如果拦截器链中间某个拦截器返回 false，则会从失败的拦截器处倒序调用 afterCompletion()。
         这个失败的拦截器 afterCompletion() 不会被调用。
         */
         if (!mappedHandler.applyPreHandle(processedRequest, response)) {  
            return;  
         }  
  
         // Actually invoke the handler.  
         // 执行目标方法，真正调用Controller中方法的地方。通过上面获取的适配器调用方法，
         mv = ha.handle(processedRequest, response, mappedHandler.getHandler());  
  
         if (asyncManager.isConcurrentHandlingStarted()) {  
            return;  
         }  
  
         applyDefaultViewName(processedRequest, mv);  
         
         // 倒序调用拦截器的 postHandle 方法
         mappedHandler.applyPostHandle(processedRequest, response, mv);  
      }  
      catch (Exception ex) {  
         dispatchException = ex;  
      }  
      catch (Throwable err) {  
         // As of 4.3, we're processing Errors thrown from handler methods as well,  
         // making them available for @ExceptionHandler methods and other scenarios.         dispatchException = new NestedServletException("Handler dispatch failed", err);  
      }  
      processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);  
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

