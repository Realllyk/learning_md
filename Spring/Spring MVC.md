
## Spirng MVC的流程

![[MVC process.webp]]

1. 用户发送请求至前端控制器 `DispatcherServlet`。
2. `DispatcherServlet` 收到请求调用处理器映射器 `HandlerMapping`。
3. 处理器映射器根据请求 URL 找到具体的处理器，生成处理器执行链 `HandlerExecutionChain`（包括处理器对象和处理器拦截器），并返回给 `DispatcherServlet`。
4. `DispatcherServlet` 根据处理器 `Handler` 获取处理器适配器 `HandlerAdapter`，执行 `HandlerAdapter` 处理一系列的操作，如：参数封装、数据格式转换、数据验证等操作。
5. 执行处理器 `Handler`（Controller，也叫页面控制器）。
6. `Handler` 执行完成返回 `ModelAndView`。
7. `HandlerAdapter` 将 `Handler` 执行结果 `ModelAndView` 返回到 `DispatcherServlet`。
8. `DispatcherServlet` 将 `ModelAndView` 传给 `ViewResolver` 视图解析器。
9. `ViewResolver` 解析后返回具体 `View`。
10. `DispatcherServlet` 对 `View` 进行渲染视图（即将模型数据 `model` 填充至视图中）。
11. `DispatcherServlet` 响应用户。



---


## Handlermapping 和 HandlerAdapter 有了解吗？

### HandlerMapping：

- **作用**：HandlerMapping 负责将请求映射到处理器（Controller）。
- **功能**：根据请求的 URL、请求参数等信息，找到处理请求的 Controller。
- **类型**：Spring 提供了多种 HandlerMapping 实现，如 `BeanNameUrlHandlerMapping`、`RequestMappingHandlerMapping` 等。
- **工作流程**：根据请求信息确定要请求的处理器（Controller）。HandlerMapping 可以根据 URL、请求参数等规则确定对应的处理器。

### HandlerAdapter：

- **作用**：HandlerAdapter 负责调用处理器（Controller）来处理请求。
- **功能**：处理器（Controller）可能有不同的接口类型（如 Controller 接口、HttpRequestHandler 接口等），HandlerAdapter 根据处理器的类型来选择合适的方法调用处理器。
- **类型**：Spring 提供了多个 HandlerAdapter 实现，用于适配不同类型的处理器。
- **工作流程**：根据处理器的接口类型，选择相应的 HandlerAdapter 来调用处理器。


### 工作流程：

1. 当客户端发送请求时，**HandlerMapping** 根据请求信息找到对应的处理器（Controller）。
2. **HandlerAdapter** 根据处理器的类型选择合适的方法来调用处理器。
3. 处理器执行相应的业务逻辑，生成 `ModelAndView`。
4. **HandlerAdapter** 将处理器的执行结果包装成 `ModelAndView`。
5. 视图解析器根据 `ModelAndView` 找到对应的视图进行渲染。
6. 将渲染后的视图返回给客户端。


**总结**：
HandlerMapping 和 HandlerAdapter 协同工作，通过请求映射到处理器，并调用处理器来处理请求，实现了请求处理的流程。它们的灵活性使得 Spring MVC 中可以支持多种处理器和处理方式，提高了框架的扩展性和适应性。

