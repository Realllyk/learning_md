# Spring AOP 核心知识总结

## 1. Advice、Advisor、MethodInterceptor 的关系

- **Advice（通知）**
  - 定义“要干的事”，横切逻辑本身。
  - 常见类型：`BeforeAdvice`、`AfterReturningAdvice`、`ThrowsAdvice`、`MethodInterceptor`。
  - `@Before`、`@After`、`@Around` 注解对应的就是 Advice。

- **Advisor（通知器）**
  - `Advisor = Pointcut + Advice`
  - 负责“在哪干”和“干什么”的绑定。
  - Spring 代理对象内部维护的其实是 **Advisor 列表**。

- **MethodInterceptor**
  - AOP 联盟（AOP Alliance）的核心接口：
    ```java
    public interface MethodInterceptor {
        Object invoke(MethodInvocation invocation) throws Throwable;
    }
    ```
  - 一种特殊的 Advice，可以完全控制方法调用过程。
  - Spring 在运行时会把各种 Advice **适配成 MethodInterceptor** 来执行。

---

## 2. Advisor 列表与 MethodInterceptor 链，以及方法调用顺序

- **持久存储**：代理对象内部持有的是 **Advisor 列表**。
  - 每个 Advisor = 切点（Pointcut）+ 通知逻辑（Advice）。

- **调用时过程**：
  1. 调用某方法时，Spring 根据方法签名遍历 Advisor。
  2. 对匹配的 Advisor，把 Advice 转换成对应的 MethodInterceptor。
  3. 从缓存里拿到该方法的 **拦截器链（MethodInterceptor[]）**。
  4. 顺序执行拦截器链 → 最终调用目标方法。

- **特点**：
  - `MethodInterceptor[]` 链在 **代理对象创建时构建并缓存**。
  - 方法调用时不会重新生成，只是执行缓存里的链。
  - 链执行顺序受 `@Order` 或 `Ordered` 控制。

**调用链示意：**

```
代理对象 (Proxy)
   └── Advisor List [持久]
         ├─ Advisor1 = Pointcut1 + Advice1
         ├─ Advisor2 = Pointcut2 + Advice2
         └─ Advisor3 = Pointcut3 + Advice3
                ↓ 匹配并适配
   └── MethodInterceptor Chain [缓存/调用时执行]
         ├─ Interceptor for Advice1
         ├─ Interceptor for Advice3
         └─ ... → 目标方法
```

---

## 3. 两种实现方式

### 3.1 基于 `@Aspect`（最常见）

```java
@Aspect
@Component
@Order(0) // 优先级高于事务
public class LoggingAspect {

    @Around("@annotation(com.example.aop.Logged)")
    public Object log(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("Before: " + pjp.getSignature());
        Object ret = pjp.proceed();
        System.out.println("After: " + pjp.getSignature());
        return ret;
    }
}
```

- 使用 `@Aspect` + 注解或表达式定义切点。
- Spring 会在启动时把 `@Aspect` 解析成多个 Advisor。
- Advice 会被适配为 MethodInterceptor，放进代理对象的拦截器链。

### 3.2 基于手写 `MethodInterceptor` / `Advisor`

```java
@Configuration
public class AopConfig {

    @Bean
    public Advisor loggingAdvisor() {
        AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
        pointcut.setExpression("execution(* com.example.service..*(..))");

        MethodInterceptor advice = invocation -> {
            System.out.println("[AOP] Before " + invocation.getMethod());
            Object ret = invocation.proceed();
            System.out.println("[AOP] After " + invocation.getMethod());
            return ret;
        };

        return new DefaultPointcutAdvisor(pointcut, advice);
    }
}
```

- 手动创建 `MethodInterceptor`（Advice 逻辑）。
- 用 `Pointcut` + `Advice` 组成 `Advisor`。
- Spring 代理会基于这些 Advisor 构建方法缓存和拦截器链。

---

## 总结

1. **Advice**：干什么。  
2. **Advisor**：在哪干 + 干什么。  
3. **MethodInterceptor**：统一执行接口，Spring 在底层把所有 Advice 适配为拦截器。  
4. **代理对象持有 Advisor 列表**，方法调用时基于 Advisor 构建并缓存 `MethodInterceptor[]` 链，直接执行。  
5. **两种写法**：
   - `@Aspect`：简洁直观，适合业务层面 AOP。
   - 手写 `MethodInterceptor/Advisor`：更底层、更灵活，适合框架化与动态控制。
