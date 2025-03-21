## Spring架构核心特性包括

- **IoC容器**: Spring通过控制反转实现了对象的创建和对象的依赖关系管理。开发者只需要定义好Bean及其依赖维件，Spring容器负责创建和组装这些对象。
- **AOP**: 面向分面编程，允许开发者定义分割互斥，例如事务管理、安全控制等，独立于业务逻辑的代码。通过AOP，可以将这些元素模块化，提高代码的可维护性和可重用性。
- **事务管理**: Spring提供了统一的事务管理接口，支持声明式和编程式事务。开发者可以轻松地进行事务管理，而无需关心具体的事务API。
- **MVC框架**: Spring MVC是一个基于Servlet API构建的Web框架，采用了模型-视图-控制器(MVC)架构。它支持活动的URL到页面控制器的映射，以及各种模板技术。<span style="color: blue;">（简化Servlet程序）</span>

![[backend_framework.png]]
![[ssm.png]]


---

## Spring IoC和AOP 区别

- **IoC**: 即控制反转的意思，它是一种创建和获取对象的技术思想，依赖注入(DI)是实现该技术的一种方式。传统开发过程中，我们需要通过new关键字来创建对象。使用IoC思想开发方式的话，我们不通过new关键字创建对象，而是通过IoC容器来帮我们实例化对象。通过IoC方式，可以大大降低对象之间的耐劣性。
- **AOP**: 是面向分面编程，能够将那些业务无关，却与业务情景所共用的逻辑封装起来，以减少系统的重复代码，降低模块之间的耐劣性。Spring AOP 就是基于动态代理的，如果要代理的对象，实现了某个接口，那么 Spring AOP 会使用 JDK Proxy 来创建代理对象。而对于没有实现接口的对象，就无法用 JDK Proxy 进行代理了，这时候 Spring AOP 会使用 Cglib 生成一个被代理对象的子类来作为代理。


---

## Spring AOP 介绍

Spring AOP是Spring架构中的一个重要模块，用于实现面向分面编程。
我们知道，Java 就是一间面向对象编程的语言，在 OOP 中最小的单元就是Class 对象，但在 AOP 中最小的单元是“切面”。“一个切面”可以封装多种类型和对象，对封进行模块化管理，例如事务管理。

在面向切面编程的思想中，把功能分为两种
- **核心业务**: 登录、注册、增删改查，都叫核心业务
- **周边业务**: 日志、事务管理这类功能称为周边业务

在面向切面编程中，核心业务功能和周边功能是分别独立进行开发，两者不是交繁的，然后把切面功能和核心业务通过“结合”进行使用，这就是AOP。
AOP能够将业务无关，却与业务情景所共用的逻辑分离起来（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，降低模块之间的耐劣性，并为从未来的可扩展性和可维护性。

### AOP 中的基本概念

- **Aspect**: 切面，只是一个概念，没有具体的接口或类之对应，是 Join point、Advice 和 Pointcut 的一个统合。
- **Join point**: 连接点，指定执行过程中的一个点，例如方法调用、异常处理等。<span style="color: orange;">在 Spring AOP 中，只支持方法级别的连接点</span>。
- **Advice**: 通知，即我们定义的一个切面中的横切逻辑，有`around` ，`before`和`after`三种类型。
- **Pointcut**: 切点，用于匹配连接点，一个 Aspect 中包含哪些 Join point 需要用 Pointcut 进行过滤。
- **Introduction**: 引入，让一个切面可以应用在没有具备实现过的实例或接口上。
- **Weaving**: 织入，指定如何将分分离逻辑插入到相应方法中，使得相应的通知逻辑在方法调用时得以执行。
- **AOP proxy**: AOP 代理，指在 AOP 实现框架中实现切面协议的对象。在 Spring AOP 中有两种代理，分别是 JDK 动态代理和 CGLIB 动态代理。
- **Target object**: 目标对象，就是被代理的对象。


---

## Spring IOC的实现机制

- **反射**: Spring IOC 容器利用 Java 的反射机制动态地加载类、创建对象实例及调用对象方法。反射允许在运行时检查类、方法、属性等信息，从而实现灵活的对象实例化和管理。
- **依赖注入**: IOC 的核心概念是依赖注入，即容器负责管理应用程序组件之间的依赖关系。Spring 通过构造函数注入、属性注入或方法注入，将组件之间的依赖关系描述在配置文件中或使用注解。
- **设计模式 - 工厂模式**: Spring IOC 容器通常采用工厂模式管理对象的创建和生命周期。容器作为工厂负责实例化 Bean 并管理它们的生命周期，将 Bean 的实例化过程交给容器管理。
- **容器实现**: Spring IOC 容器是实现 IOC 的核心，通常使用 BeanFactory 或 ApplicationContext 来管理 Bean。BeanFactory 是 IOC 容器的基本形式，提供基本的 IOC 功能；ApplicationContext 是 BeanFactory 的扩展，并提供更多企业级功能。


---


## Spring AOP 实现机制

Spring AOP 的实现依赖于 **动态代理技术**。动态代理是在运行时动态生成代理对象，而不是在编译时。它允许开发者在运行时指定要代理的接口和行为，从而实现在不修改源码的情况下增强方法的功能。

Spring AOP 支持两种动态代理：
- **基于JDK的动态代理**: 使用 `java.lang.reflect.Proxy` 类和`java.lang.reflect.InvocationHandler` 接口实现。这种方式需要代理的类实现一个或多个接口。
- **基于CGLIB的动态代理**: 当被代理的类没有实现接口时，Spring 会使用 CGLIB 库生成一个被代理类的子类作为代理。CGLIB（Code Generation Library）是一个第三方代码生成库，通过继承方式实现代理。



---


## 如何理解Spring IOC

**IOC**：Inversion Of Control，即控制反转，是一种设计思想。在传统的 Java SE 程序设计中，我们直接在对象内部通过 `new` 的方式来创建对象，是程序主动创建依赖对象；

![[jave_se_control.png]]

而在Spirng程序设计中，IOC是由专门的容器去控制对象。
![[INVERSE_CONTROL.png]]

### **总结**
IOC 解决了繁琐的对象生命周期的操作，解耦了我们的代码。**所谓反转**：其实是反转的控制权，前面提到是由 Spring 来控制对象的生命周期，那么对象的控制就完全脱离了我们的控制，控制权交给了 Spring。这个反转指的是：我们由对象的控制者变成了 IOC 的被动控制者。


---


## 依赖倒置、依赖注入、控制反转分别是什么？

- **控制反转**："控制" 指的是对程序执行流程的控制，而 "反转" 指的是在没有使用框架之前，程序员自己控制整个程序的执行。<span style="color: blue;">在使用框架之后，整个程序的执行流程通过框架来控制。</span>流程的控制权从程序员 "反转" 给了框架。
- **依赖注入**：依赖注入和控制反转恰恰相反，它是一种具体的编码技巧。我们不通过 `new` 的方式在类内部创建依赖类的对象，而是<span style="color: blue;">将依赖的类对象在外部创建好之后</span>，通过构造函数、函数参数等方式传递（或注入）给类来使用。
- **依赖倒置(OOP的一种设计模式)**：这一原则跟控制反转有点类似，主要用来指导框架层面的设计。<span style="color: blue;">高层模块不依赖低层模块，它们共同依赖同一个抽象</span>。抽象不依赖具体实现细节，具体实现细节依赖抽象。


---


## 动态代理和静态代理的区别

代理是一种常用的设计模式，目的是：<span style="color: blue;">为其他对象提供一个代理以控制对某个对象的访问</span>，将两个类的关系解耦。代理类和委托类都要实现相同的接口，因为代理真正调用的是委托类的方法。
### 区别：
- **静态代理**：由程序员创建或者由特定工具创建，在代码<span style="color: blue;">编译时就确定了</span>被代理的类是一个静态代理。静态代理通常只代理一个类；
- **动态代理**：在<span style="color: blue;">代码运行期间</span>，运用反射机制动态创建生成。动态代理代理的是<span style="color: blue;">一个接口下的多个实现类</span>。


---


## 反射

反射机制是指程序在运行状态下，对于任意一个类，都能够获取这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意属性和方法。也就是说，Java 反射允许在运行时获取类的信息并动态操作对象，即使在编译时不知道具体的类也能实现。

### 反射的特点：

1. **运行时类信息访问**：反射机制允许程序在运行时获取类的完整结构信息，包括类名、包名、父类、实现的接口、构造函数、方法和字段等。
2. **动态对象创建**：可以使用反射 API 动态地创建对象实例，即使在编译时不知道具体的类名。这是通过 `Class` 类的 `newInstance()` 方法或 `Constructor` 对象的 `newInstance()` 方法实现。
3. **动态方法调用**：可以在运行时动态地调用对象的方法，包括私有方法。这是通过 `Method` 类的 `invoke()` 方法实现，允许你传入对象实例和参数执行目标方法。
4. **访问和修改字段**：反射允许在程序运行时访问和修改对象的字段值，即使是私有的。这是通过 `Field` 类的 `get()` 和 `set()` 方法完成的。


---


## Spring 是如何解决循环依赖的？

循环依赖指的是两个类中的属性相互依赖对方。例如 A 类中有 B 属性，B 类中有 A 属性，从而形成了一个依赖闭环，如下图。

```java
class BeanA {
    @Autowired
    private BeanB b;
}

class BeanB {
    @Autowired
    private BeanA a;
}
```

![[loop_dependency.png]]
### **循环依赖问题在 Spring 中主要有三种情况**：

1. **第一种**：通过**构造方法**进行依赖注入时产生的循环依赖问题。
2. **第二种**：通过 **setter 方法** 进行依赖注入且是在 **多例（prototype）模式** 下产生的循环依赖问题。
3. **第三种**：通过 **setter 方法** 进行 依赖注入且是在 **单例（singleton）模式** 下产生的循环依赖问题。

###  **这三种循环依赖的区别**
1. **构造方法注入的循环依赖（无法解决）**
    - **问题**：如果 A 需要 B 的实例，B 也需要 A 的实例，并且两者都是通过 **构造函数** 进行依赖注入，那么 Spring **无法** 创建它们，因为在实例化一个 Bean 之前需要另一个 Bean 先被实例化，形成**死锁**。
    - **解决方案**：Spring **无法** 直接解决这个问题，通常的做法是改为 **setter 注入** 或 **使用 `@Lazy` 延迟加载**。
2. **多例（Prototype）模式的 setter 依赖注入（无法解决）**
    - **问题**：在 **Prototype 模式** 下，<span style="color: orange;">每次请求 Bean 时都会创建新的实例</span>，Spring **不会** 将它们放入缓存，也不会进行提前曝光，因此它们的依赖关系无法提前解析，从而导致 **循环依赖无法解决**。
    - **解决方案**：
        - 不能依赖 Spring 自己的循环依赖解决方案，改用 **手动创建 Bean**（如 `@Bean` 方法中手动 `new`）。
        - 通过 **代理模式** 或 **第三方依赖管理**（如 `ObjectFactory`）。
3. **单例（Singleton）模式的 setter 依赖注入（Spring 可以自动解决）**
    - **问题**：在 **单例模式** 下，Spring 会创建一个 Bean **但不会立刻初始化**，而是**先把它的引用暴露到缓存（`singletonFactories`）中**，这样当依赖它的 Bean 需要它时，Spring 就可以获取到一个**尚未完全初始化的 Bean**，从而**避免死循环**。
    - **解决方案**：
        - **Spring 自动解决**，因为 Spring 采用**三级缓存机制**（`singletonObjects`、`earlySingletonObjects` 和 `singletonFactories`）来存储 Bean 实例，确保可以先注入一个代理对象，避免死锁问题。

### **总结**

|**循环依赖类型**|**是否可解决**|**Spring 解决方式**|**解决方案**|
|---|---|---|---|
|**构造方法注入**|❌ **无法解决**|不支持|改为 `setter` 注入 或 `@Lazy` 延迟加载|
|**多例（Prototype）模式**|❌ **无法解决**|不支持|需手动创建 Bean 或使用代理模式|
|**单例（Singleton）模式**|✅ **可解决**|**Spring 三级缓存**|Spring 自动解决，无需修改|



---


## Spring AOP 体现的是代理模式还是装饰器模式
### ✅ 结论先行：

> **Spring AOP 在实现层面上用的是代理模式（Proxy Pattern），但它的本质增强方式非常符合装饰器（包装器）模式的思想。**

也就是说：

|层面|使用的是什么|
|---|---|
|**实现手段**|✅ 是代理模式（JDK 动态代理 或 CGLIB）|
|**设计思想**|✅ 是装饰器模式（通过包裹原方法，实现前后增强）|


### 🔍 那我们怎么理解这个区别？

### 🧩 一、代理模式 vs 装饰器模式的区别

|对比点|代理模式（Proxy）|装饰器模式（Decorator）|
|---|---|---|
|目的|控制对目标对象的访问|动态地扩展对象功能|
|是否增强原对象|不一定，主要用于**访问控制**|✅ 增强原对象功能|
|是否与目标类实现相同接口|✅ 是|✅ 是|
|是否可嵌套组合|一般不强调|✅ 强调可“包一层又一层”|


### 📦 二、AOP 中的代理做了什么？

假设我们有一个目标方法：
```java
public class UserService {
    public void saveUser() {
        System.out.println("保存用户");
    }
}
```


如果你加了一个切面（Aspect）：
```java
@Before("execution(* saveUser(..))")
public void logBefore() {
    System.out.println("日志记录：saveUser 被调用");
}

```


Spring 会为这个类生成一个代理类：
```java
public class UserServiceProxy extends UserService {
    @Override
    public void saveUser() {
        System.out.println("日志记录：saveUser 被调用"); // 前置增强
        super.saveUser();                             // 调用目标方法
    }
}
```

### 所以说：

> Spring AOP 用 **代理** 来实现 “方法增强”，这个增强的方式 ——“前后包一层”的思路，就是**装饰器模式的体现**。


### 🧠 用一句话区分：

> - Spring AOP **用代理来实现功能包装**
>     
> - 包装的结构方式体现了 **装饰器（包装器）模式的思想**



---


## @Configuration 和 @Component的区别

在Spring中，@Configuration定义配置类的作用是什么？是代表配置类中带有@Bean注解的方法的返回值都会被IOC容器管理吗

`@Configuration` 是一个**标记配置类**的注解，表示这个类中可能包含一个或多个 `@Bean` 方法，这些方法的返回值将被注册为 **Spring 容器中的 Bean**。


### 🧠 你说的这句话：

> 是代表配置类中带有 @Bean 注解的方法的返回值都会被 IOC 容器管理吗？

✅ **答案是：是的！**

如果这个类被 `@Configuration` 注解标注，那么它里面所有带 `@Bean` 注解的方法返回的对象，都会被当作 Bean 注册到 Spring 容器中，生命周期由 Spring 管理。


### 🔍 举个例子：

```java
@Configuration
public class AppConfig {

    @Bean
    public UserService userService() {
        return new UserService();
    }
}

```


### ❗ 补充重点：`@Configuration` vs 普通类 + `@Bean`

Spring 还允许你在没有 `@Configuration` 的类里写 `@Bean`，但是行为上有区别！
```java
@Component
public class SomeConfig {
    @Bean
    public MyBean myBean() {
        return new MyBean();
    }
}
```

这种写法**也会注册 Bean**，但它不具备 **完整的代理增强能力**。举个关键区别：
#### ✅ 带 `@Configuration` 的类：
```java
@Configuration
public class MyConfig {

    @Bean
    public A a() {
        return new A(b());
    }

    @Bean
    public B b() {
        return new B();
    }
}
```

在这种情况下，Spring 会用 **CGLIB** 创建配置类的代理对象，保证 `b()` 方法调用的是 Spring 容器中的单例 Bean。


#### ❌ 不带 `@Configuration`（比如只加了 `@Component`）：

```java
@Component
public class MyConfig {

    @Bean
    public A a() {
        return new A(b());  // ⚠️ 此处调用的是当前类的 b() 方法，不是容器中的那个 b()
    }

    @Bean
    public B b() {
        return new B();
    }
}

```

这种写法中 `a()` 方法中调用 `b()` 实际是新创建一个对象，不是从容器里取，**可能导致 Bean 重复创建、不走容器管理**，这就是“没有代理”的区别。


### ✅ 总结

| 注解                     | 是否注册 Bean | 是否有代理增强         | 推荐用法 |
| ---------------------- | --------- | --------------- | ---- |
| `@Configuration`       | ✅ 是       | ✅ 有 CGLIB 代理，推荐 | ✅✅✅  |
| `@Component` + `@Bean` | ✅ 是       | ❌ 无代理，方法间调用不走容器 | ❌ 避免 |
|                        |           |                 |      |

**配置类**是否被代理增强，直接决定了 @Bean 方法之间互相调用时是否从 Spring 容器中获取已存在的单例 Bean。
```java
@Configuration
public class MyConfig {

    @Bean
    public A a() {
        return new A(b()); // 调用 b()
    }

    @Bean
    public B b() {
        return new B();
    }
}
```

#### 如果没有被代理（比如只加了 `@Component`）：
- `a()` 方法中直接调用当前类的 `b()` 方法（普通 Java 方法调用）。
- 每次执行 `b()` 就会 new 一个新的 B 实例。
- 你以为你用了 `@Bean` 得到了容器中的东西，其实根本不是！