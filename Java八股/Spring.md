# Spring

## 核心特性

Spring框架核心特性包括：

- **IoC容器**：Inversion of Control，Spring通过控制反转实现了对象的创建和对象间的依赖关系管理。开发者只需要定义好Bean及其依赖关系，Spring容器负责创建和组装这些对象。
- **AOP**：Aspect Orient Programming，面向切面编程，允许开发者定义横切关注点，例如事务管理、安全控制等，独立于业务逻辑的代码。通过AOP，可以将这些关注点模块化，提高代码的可维护性和可重用性。
- **事务管理**：Spring提供了一致的事务管理接口，支持声明式和编程式事务。开发者可以轻松地进行事务管理，而无需关心具体的事务API。
- **MVC框架**：Model View Controller，Spring MVC是一个基于Servlet API构建的Web框架，采用了模型-视图-控制器（MVC）架构。它支持灵活的URL到页面控制器的映射，以及多种视图技术。

<div align="center"> <img src="https://oss.javaguide.cn/github/javaguide/jvme0c60b4606711fc4a0b6faf03230247a.png" width="75%"/> </div>

## IoC

IoC （Inversion of Control ）即控制反转/反转控制。它是一种思想不是一个技术实现。描述的是：Java 开发领域对象的创建以及管理的问题。将对象的创建、组装、管理控制权从应用程序的显式代码转移到外部IoC容器，由容器负责Bean的完整生命周期。

**为什么叫控制反转?**

- **控制** ：指的是对象创建（实例化、管理）的权力
- **反转** ：控制权交给外部环境（IoC 容器）

**IoC 是一种设计思想，依赖注入（DI）是 IoC 的一种具体实现方式，但不是唯一方式。**

因为 **IoC 还可以通过其他方式实现**，DI 只是最常见的一种：

| 实现方式                | 说明                     | 示例                                 |
| :---------------------- | :----------------------- | :----------------------------------- |
| **依赖注入（DI）**      | 容器主动把依赖传给对象   | Spring `@Autowired`                  |
| **依赖查找（DL）**      | 对象主动从容器中查找依赖 | `ApplicationContext.getBean()`、JNDI |
| **策略模式 + 配置文件** | 通过外部配置决定具体实现 | 早期的 Struts、XML 配置              |



### 依赖注入

将对象的创建和依赖关系的管理交给 Spring 容器来完成，类只需要声明自己所依赖的对象，容器会在运行时将这些依赖对象注入到类中，从而降低了类与类之间的耦合度，提高了代码的可维护性和可测试性。

常见的依赖注入的实现方式：

* 构造器注入
* Setter 方法注入
* 字段注入



### 从哪些方面设计一个Spring IoC

- Bean的生命周期管理：需要设计Bean的创建、初始化、销毁等生命周期管理机制，可以考虑使用工厂模式和单例模式来实现。
- 依赖注入：需要实现依赖注入的功能，包括属性注入、构造函数注入、方法注入等，可以考虑使用反射机制和XML配置文件来实现。
- Bean的作用域：需要支持多种Bean作用域，比如单例、原型、会话、请求等，可以考虑使用Map来存储不同作用域的Bean实例。
- AOP功能的支持：需要支持AOP功能，可以考虑使用动态代理机制和切面编程来实现。
- 异常处理：需要考虑异常处理机制，包括Bean创建异常、依赖注入异常等，可以考虑使用try-catch机制来处理异常。
- 配置文件加载：需要支持从不同的配置文件中加载Bean的相关信息，可以考虑使用XML、注解或者Java配置类来实现。



## AOP

AOP（面向切面编程）是核心特性之一，它通过分离横切关注点（如日志、事务、安全等）与业务逻辑，让代码更清晰、可维护。

| 术语              | 含义                                                         |
| ----------------- | ------------------------------------------------------------ |
| 目标(Target)      | 被通知的对象                                                 |
| 代理(Proxy)       | 向目标对象应用通知之后创建的代理对象                         |
| 连接点(JoinPoint) | 目标对象的所属类中，定义的所有方法均为连接点                 |
| 切入点(Pointcut)  | 被切面拦截 / 增强的连接点（切入点一定是连接点，连接点不一定是切入点） |
| 通知(Advice)      | 增强的逻辑 / 代码，也即拦截到目标对象的连接点之后要做的事情  |
| 切面(Aspect)      | 切入点(Pointcut)+通知(Advice)                                |
| 织入(Weaving)     | 将通知应用到目标对象，进而生成代理对象的过程动作             |

### 实现原理

Spring AOP的实现依赖于**动态代理技术**。动态代理是在运行时动态生成代理对象，而不是在编译时。它允许开发者在运行时指定要代理的接口和行为，从而实现在不修改源码的情况下增强方法的功能。

Spring AOP支持两种动态代理：

- **基于JDK的动态代理**：使用java.lang.reflect.Proxy类和java.lang.reflect.InvocationHandler接口实现。这种方式需要代理的类实现一个或多个接口。
- **基于CGLIB的动态代理**：当被代理的类没有实现接口时，Spring会使用CGLIB库生成一个被代理类的子类作为代理。CGLIB（Code Generation Library）是一个第三方代码生成库，通过继承方式实现代理。

### 常用注解

常用的注解包括：

- @Aspect：用于定义切面，标注在切面类上。
- @Pointcut：定义切点，标注在方法上，用于指定连接点。
- @Before：在方法执行之前执行通知。
- @After：在方法执行之后执行通知（无论方法正常返回还是抛出异常）。
- @Around：在方法执行前后都执行通知，是功能最强的一种。
- @AfterReturning：在方法正常返回结果后执行通知。
- @AfterThrowing：在方法抛出异常后执行通知。

<div align="center"> <img src="https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/aspectj-advice-types.jpg" width="50%"/> </div>





## Bean

### 生命周期

总共分为四步：实例化 —> 属性赋值 —> 初始化 —> 销毁。

1. **创建 Bean 的实例**：Bean 容器首先会找到配置文件中的 Bean 定义，然后使用 Java 反射 API 来创建 Bean 的实例。
2. **Bean 属性赋值/填充**：为 Bean 设置相关属性和依赖，例如`@Autowired` 等注解注入的对象、`@Value` 注入的值、`setter`方法或构造函数注入依赖和值、`@Resource`注入的各种资源。
3. **Bean 初始化**：
   - 如果 Bean 实现了 `BeanNameAware` 接口，调用 `setBeanName()`方法，传入 Bean 的名字。
   - 如果 Bean 实现了 `BeanClassLoaderAware` 接口，调用 `setBeanClassLoader()`方法，传入 `ClassLoader`对象的实例。
   - 如果 Bean 实现了 `BeanFactoryAware` 接口，调用 `setBeanFactory()`方法，传入 `BeanFactory`对象的实例。
   - 与上面的类似，如果实现了其他 `*.Aware`接口，就调用相应的方法。
   - 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessBeforeInitialization()` 方法
   - 如果 Bean 实现了`InitializingBean`接口，执行`afterPropertiesSet()`方法。
   - 如果 Bean 在配置文件中的定义包含 `init-method` 属性，执行指定的方法。
   - 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessAfterInitialization()` 方法。==**AOP生成的代理对象发生在这一步**==
4. **销毁 Bean**：销毁并不是说要立马把 Bean 给销毁掉，而是把 Bean 的销毁方法先记录下来，将来需要销毁 Bean 或者销毁容器的时候，就调用这些方法去释放 Bean 所持有的资源。
   - 如果 Bean 实现了 `DisposableBean` 接口，执行 `destroy()` 方法。
   - 如果 Bean 在配置文件中的定义包含 `destroy-method` 属性，执行指定的 Bean 销毁方法。或者，也可以直接通过`@PreDestroy` 注解标记 Bean 销毁之前执行的方法。

<div align="center"> <img src="https://oss.javaguide.cn/github/javaguide/system-design/framework/spring/spring-bean-lifestyle.png" width="100%"/> </div>



### Bean是否单例？

Spring 中的 Bean 默认都是单例的。

就是说，每个Bean的实例只会被创建一次，并且会被存储在Spring容器的缓存中，以便在后续的请求中重复使用。这种单例模式可以提高应用程序的性能和内存效率。

但是，Spring也支持将Bean设置为多例模式，即每次请求都会创建一个新的Bean实例。要将Bean设置为多例模式，可以在Bean定义中通过设置scope属性为"prototype"来实现。

需要注意的是，虽然Spring的默认行为是将Bean设置为单例模式，但在一些情况下，使用多例模式是更为合适的，例如在创建状态不可变的Bean或有状态Bean时。此外，需要注意的是，如果Bean单例是有状态的，那么在使用时需要考虑线程安全性问题。



## 三级缓存

### 循环依赖问题

循环依赖指的是两个类中的属性相互依赖对方：例如 A 类中有 B 属性，B 类中有 A属性，从而形成了一个依赖闭环。

循环依赖问题在Spring中主要有三种情况：

- 第一种：通过构造方法进行依赖注入时产生的循环依赖问题。
- 第二种：通过setter方法进行依赖注入且是在多例（原型）模式下产生的循环依赖问题。
- 第三种：通过 setter 方法或字段（Field）进行依赖注入且是在单例模式下产生的循环依赖问题。

只有【第三种方式】的循环依赖问题被 Spring 解决了，其他两种方式在遇到循环依赖问题时，Spring都会产生异常。

### 三级缓存结构

Spring 在 `DefaultSingletonBeanRegistry` 类中维护了三个重要的缓存 (Map)，称为“三级缓存”：

- `singletonObjects` (一级缓存)：存放的是完全初始化好的、可用的 Bean 实例，`getBean()` 方法最终返回的就是这里面的 Bean。此时 Bean 已实例化、属性已填充、初始化方法已执行、AOP 代理（如果需要）也已生成。==成品对象==
- `earlySingletonObjects` (二级缓存)：存放的是提前暴露的 Bean 的原始对象引用 或 早期代理对象引用，专门用来处理循环依赖。当一个 Bean 还在创建过程中（尚未完成属性填充和初始化），但它的引用需要被注入到另一个 Bean 时，就暂时放在这里。此时 Bean 已实例化（调用了构造函数），但属性尚未填充，初始化方法尚未执行，它可能是一个原始对象，也可能是一个为了解决 AOP 代理问题而提前生成的代理对象。==半成品对象==
- `singletonFactories` (三级缓存)：存放的是 Bean 的 `ObjectFactory` 工厂对象。，这是解决循环依赖和 AOP 代理协同工作的关键。当 Bean 被实例化后（刚调完构造函数），Spring 会创建一个 `ObjectFactory` 并将其放入三级缓存。这个工厂的 `getObject()` 方法负责返回该 Bean 的早期引用（可能是原始对象，也可能是提前生成的代理对象），当检测到循环依赖需要注入一个尚未完全初始化的 Bean 时，就会调用这个工厂来获取早期引用。==工厂对象==

### 具体步骤

假设存在两个相互依赖的单例Bean：`BeanA` 依赖 `BeanB`，同时 `BeanB` 也依赖 `BeanA`。当Spring容器启动时，它会按照以下流程处理：

- 第一步：创建`BeanA`的实例并提前暴露工厂。

Spring首先调用`BeanA`的构造函数进行实例化，此时得到一个原始对象（尚未填充属性）。紧接着，Spring会将一个特殊的`ObjectFactory`工厂对象存入第三级缓存（`singletonFactories`）。这个工厂的使命是：当其他Bean需要引用`BeanA`时，它能动态返回当前这个半成品的`BeanA`（可能是原始对象，也可能是为应对AOP而提前生成的代理对象）。此时`BeanA`的状态是"已实例化但未初始化"，像一座刚搭好钢筋骨架的大楼。

- 第二步：填充`BeanA`的属性时触发`BeanB`的创建。

Spring开始为`BeanA`注入属性，发现它依赖`BeanB`。于是容器转向创建`BeanB`，同样先调用其构造函数实例化，并将`BeanB`对应的`ObjectFactory`工厂存入三级缓存。至此，三级缓存中同时存在`BeanA`和`BeanB`的工厂，它们都代表未完成初始化的半成品。

- 第三步：`BeanB`属性注入时发现循环依赖。

当Spring试图填充`BeanB`的属性时，检测到它需要注入`BeanA`。此时容器启动依赖查找：

1. 在一级缓存（存放完整Bean）中未找到`BeanA`；
2. 在二级缓存（存放已暴露的早期引用）中同样未命中；
3. 最终在三级缓存中定位到`BeanA`的工厂。

Spring立即调用该工厂的`getObject()`方法。这个方法会执行关键决策：若`BeanA`需要AOP代理，则动态生成代理对象（即使`BeanA`还未初始化）；若无需代理，则直接返回原始对象。得到的这个早期引用（可能是代理）被放入二级缓存（`earlySingletonObjects`），同时从三级缓存清理工厂条目。最后，Spring将这个早期引用注入到`BeanB`的属性中。至此，`BeanB`成功持有`BeanA`的引用——尽管`BeanA`此时仍是个半成品。

- 第四步：完成`BeanB`的生命周期。

`BeanB`获得所有依赖后，Spring执行其初始化方法（如`@PostConstruct`），将其转化为完整可用的Bean。随后，`BeanB`被提升至一级缓存（`singletonObjects`），二级和三级缓存中关于`BeanB`的临时条目均被清除。此时`BeanB`已准备就绪，可被其他对象使用。

- 第五步：回溯完成`BeanA`的构建。

随着`BeanB`创建完毕，流程回溯到最初中断的`BeanA`属性注入环节。Spring将已完备的`BeanB`实例注入`BeanA`，接着执行`BeanA`的初始化方法。这里有个精妙细节：若之前为`BeanA`生成过早期代理，Spring会直接复用二级缓存中的代理对象作为最终Bean，而非重复创建。最终，完全初始化的`BeanA`（可能是原始对象或代理）入驻一级缓存，其早期引用从二级缓存移除。至此循环闭环完成，两个Bean皆可用。

三级缓存的设计的精髓：

- **三级缓存工厂**（`singletonFactories`）负责在实例化后立刻暴露对象生成能力，兼顾AOP代理的提前生成；
- **二级缓存**（`earlySingletonObjects`）临时存储已确定的早期引用，避免重复生成代理；
- **一级缓存**（`singletonObjects`）最终交付完整Bean。

整个机制通过**中断初始化流程、逆向注入半成品、延迟代理生成**三大策略，将循环依赖的死结转化为有序的接力协作。

值得注意的是，此方案仅适用于Setter/Field注入的**单例Bean**；构造器注入因必须在实例化前获得依赖，仍会导致无解的死锁



### 为什么用3级缓存解决循环依赖问题？用2级缓存不行吗

**三级缓存不是用来“解决循环依赖”的，而是用来“在循环依赖发生时，保证注入的对象是最终形态（代理/原始）”的。**
二级缓存缺乏生成代理的决策能力，所以必须引入三级工厂。

Spring 必须用三级缓存解决循环依赖，核心是为了正确处理需要 AOP 代理的 Bean。如果只用二级缓存，会导致注入的对象形态错误，甚至破坏单例原则。

举个例子：假设 Bean A 依赖 B，B 又依赖 A，且 A 需要被动态代理（比如加了 @Transactional）。如果只有二级缓存，当 B 创建时去注入 A，拿到的是 A 的原始对象。但 A 在后续初始化完成后才会生成代理对象，结果就是：B 拿着原始对象 A，而 Spring 容器里存的是代理对象 A —— 同一个 Bean 出现了两个不同实例，这直接违反了单例的核心约束。

三级缓存中的 ObjectFactory 就是解决这个问题的关键。它不是直接缓存对象，而是存了一个能生产对象的工厂。当发生循环依赖时，调用这个工厂的 getObject() 方法，这时 Spring 会智能判断：如果这个 Bean 最终需要代理，就提前生成代理对象并放入二级缓存；如果不需要代理，就返回原始对象。这样一来，B 注入的 A 就是最终形态（可能是代理对象），后续 A 初始化完成后也不会再创建新代理，保证了对象全局唯一。

简单说，三级缓存的本质是 “按需延迟生成正确引用” 。它既维持了 Bean 生命周期的完整性（正常流程在初始化后生成代理），又在循环依赖时特殊处理，避免逻辑矛盾。而二级缓存缺乏这种动态决策能力，因此无法替代三级缓存。 



## SpringMVC

### 处理流程

<div align="center"> <img src="https://cdn.xiaolincoding.com//picgo/1716791047520-ac0d9673-be0a-4005-8732-30bdedc8f1af.webp" width="80%"/> </div>

Spring MVC的工作流程如下：

1. 用户发送请求至前端控制器DispatcherServlet
2. DispatcherServlet收到请求调用处理器映射器HandlerMapping。
3. 处理器映射器根据请求url找到具体的处理器，生成处理器执行链HandlerExecutionChain(包括处理器对象和处理器拦截器)一并返回给DispatcherServlet。
4. DispatcherServlet根据处理器Handler获取处理器适配器HandlerAdapter执行HandlerAdapter处理一系列的操作，如：参数封装，数据格式转换，数据验证等操作
5. 执行处理器Handler(Controller，也叫页面控制器)。
6. Handler执行完成返回ModelAndView
7. HandlerAdapter将Handler执行结果ModelAndView返回到DispatcherServlet
8. DispatcherServlet将ModelAndView传给ViewReslover视图解析器
9. ViewReslover解析后返回具体View
10. DispatcherServlet对View进行渲染视图（即将模型数据model填充至视图中）。
11. DispatcherServlet响应用户。



### HandlerMapping

**根据当前请求，找到对应的处理器（Handler）。**这里的 Handler 在 Spring MVC 中通常就是指 `@Controller` 中的 `@RequestMapping` 方法，而不是完整的 Controller 对象。

**核心方法**

`getHandler(HttpServletRequest request)` → 返回 `HandlerExecutionChain`（包含 Handler 和拦截器列表）。

**工作流程**

1. 接收请求
2. 根据 URL/参数等匹配规则
3. 找到对应的 Handler 和拦截器链
4. 返回给 `DispatcherServlet`



### HandlerAdapter

**调用具体处理器（Handler）来执行请求。**因为 Handler 有不同形态（`@Controller` 方法、`HttpRequestHandler`、`Servlet` 等），HandlerAdapter 负责**统一调用接口**，屏蔽差异。

**核心方法**

- `supports(Object handler)`：判断是否支持该 Handler
- `handle(HttpServletRequest, HttpServletResponse, Object handler)`：执行 Handler，返回 `ModelAndView`

**工作流程**

1. `DispatcherServlet` 调用 `handle()`
2. HandlerAdapter 执行 Handler 的业务逻辑
3. 返回 `ModelAndView`（或 `null`，如 `@ResponseBody` 直接写响应）



> **HandlerMapping 负责“找到哪个方法执行”**
> **HandlerAdapter 负责“怎么执行这个方法”**

它们的配合让 Spring MVC 可以支持多种不同类型的 Handler（注解方法、旧接口、Servlet 等），而 `DispatcherServlet` 只需要依赖这两个接口，无需关心具体实现。



## SpringBoot

### 相比于Spring的优势

- Spring Boot 提供了自动化配置，大大简化了项目的配置过程。通过约定优于配置的原则，很多常用的配置可以自动完成，开发者可以专注于业务逻辑的实现。
- Spring Boot 提供了快速的项目启动器，通过引入不同的 Starter，可以快速集成常用的框架和库（如数据库、消息队列、Web 开发等），极大地提高了开发效率。
- Spring Boot 默认集成了多种内嵌服务器（如Tomcat、Jetty、Undertow），无需额外配置，即可将应用打包成可执行的 JAR 文件，方便部署和运行。



### 设计模式

- **代理模式**：Spring 的 AOP 通过动态代理实现方法级别的切面增强，有静态和动态两种代理方式，采用动态代理方式。
- **策略模式**：Spring AOP 支持 JDK 和 Cglib 两种动态代理实现方式，通过策略接口和不同策略类，运行时动态选择，其创建一般通过工厂方法实现。
- **装饰器模式**：Spring 用 TransactionAwareCacheDecorator 解决缓存与数据库事务问题增加对事务的支持。
- **单例模式**：Spring Bean 默认是单例模式，通过单例注册表（如 HashMap）实现。
- **简单工厂模式**：Spring 中的 BeanFactory 是简单工厂模式的体现，通过工厂类方法获取 Bean 实例。
- **工厂方法模式**：Spring中的 FactoryBean 体现工厂方法模式，为不同产品提供不同工厂。
- **观察者模式**：Spring 观察者模式包含 Event 事件、Listener 监听者、Publisher 发送者，通过定义事件、监听器和发送者实现，观察者注册在 ApplicationContext 中，消息发送由 ApplicationEventMulticaster 完成。
- **模板模式**：Spring Bean 的创建过程涉及模板模式，体现扩展性，类似 Callback 回调实现方式。
- **适配器模式**：Spring MVC 中针对不同方式定义的 Controller，利用适配器模式统一函数定义，定义了统一接口 HandlerAdapter 及对应适配器类。



### SpringBoot 过滤器和拦截器

在 Spring Boot 中，过滤器（Filter）和拦截器（Interceptor）是用于处理请求和响应的两种不同机制。

| **特性**         | **过滤器（Filter）**                  | **拦截器（Interceptor）**                                    |
| ---------------- | ------------------------------------- | ------------------------------------------------------------ |
| **规范/框架**    | Servlet规范（`javax.servlet.Filter`） | Spring MVC框架（`org.springframework.web.servlet.HandlerInterceptor`） |
| **作用范围**     | 全局（所有请求、静态资源）            | Controller层（仅拦截Spring管理的请求）                       |
| **执行顺序**     | 在Servlet之前执行                     | 在DispatcherServlet之后、Controller方法前后执行              |
| **依赖注入支持** | 无法直接注入Spring Bean（需间接获取） | 支持自动注入Spring Bean                                      |
| **触发时机**     | `doFilter()`在请求前/响应后被调用     | `preHandle`、`postHandle`、`afterCompletion`分阶段触发       |
| **适用场景**     | 全局请求处理（编码、日志、安全）      | 业务逻辑相关的处理（权限、参数校验）                         |

过滤器是 Java Servlet 规范中的一部分，它可以对进入 Servlet 容器的请求和响应进行预处理和后处理。过滤器通过实现 `javax.servlet.Filter` 接口，并重写其中的 `init`、`doFilter` 和 `destroy` 方法来完成相应的逻辑。当请求进入 Servlet 容器时，会按照配置的顺序依次经过各个过滤器，然后再到达目标 Servlet 或控制器；响应返回时，也会按照相反的顺序再次经过这些过滤器。

拦截器是 Spring 框架提供的一种机制，它可以对控制器方法的执行进行拦截。拦截器通过实现 `org.springframework.web.servlet.HandlerInterceptor` 接口，并重写其中的 `preHandle`、`postHandle` 和 `afterCompletion` 方法来完成相应的逻辑。当请求到达控制器时，会先经过拦截器的 `preHandle` 方法，如果该方法返回 `true`，则继续执行后续的控制器方法和其他拦截器；在控制器方法执行完成后，会调用拦截器的 `postHandle` 方法；最后，在请求处理完成后，会调用拦截器的 `afterCompletion` 方法。



## 其他

### 常用设计模式

- **工厂设计模式** : Spring使用工厂模式通过 BeanFactory、ApplicationContext 创建 bean 对象。
- **代理设计模式** : Spring AOP 功能的实现。
- **单例设计模式** : Spring 中的 Bean 默认都是单例的。
- **模板方法模式** : Spring 中 jdbcTemplate、hibernateTemplate 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
- **包装器设计模式** : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
- **观察者模式:** Spring 事件驱动模型就是观察者模式很经典的一个应用。
- **适配器模式** :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配Controller。



### 常用注解

* @Autowired 注解

@Autowired：主要用于自动装配bean。当Spring容器中存在与要注入的属性类型匹配的bean时，它会自动将bean注入到属性中。

* @Component

这个注解用于标记一个类作为Spring的bean。当一个类被@Component注解标记时，Spring会将其实例化为一个bean，并将其添加到Spring容器中。

* @Configuration

@Configuration，注解用于标记一个类作为Spring的配置类。配置类可以包含@Bean注解的方法，用于定义和配置bean，作为全局配置。

* @Bean

@Bean注解用于标记一个方法作为Spring的bean工厂方法。当一个方法被@Bean注解标记时，Spring会将该方法的返回值作为一个bean，并将其添加到Spring容器中。

* @Service

@Service，这个注解用于标记一个类作为服务层的组件。它是@Component注解的特例，用于标记服务层的bean，一般标记在业务service的实现类。

* @Controller

@Controller注解用于标记一个类作为控制层的组件。它也是@Component注解的特例，用于标记控制层的bean。这是MVC结构的另一个部分，加在控制层。



### 什么情况下事务会失效

1. **异常被 try-catch 吞掉**

- **现象**：方法内 catch 了异常且没有重新抛出，Spring 代理不知情 → 事务正常提交，不回滚。
- **纠正**：要么在 catch 中手动回滚（`TransactionAspectSupport.currentTransactionStatus().setRollbackOnly()`），要么重新抛出异常。

2. **抛出的是受检异常（Checked Exception）**

- **默认行为**：只对 `RuntimeException` 及 `Error` 回滚
- **解决方案**：`@Transactional(rollbackFor = Exception.class)`
- **注意**：`SQLException` 虽然是受检异常，但在 JDBC 操作中常被 Spring 转为 `DataAccessException`（非受检），所以不一定失效，但不能依赖。

3. **同类内部方法调用（this 调用）**

```java
@Service
public class UserService {
    @Transactional
    public void methodA() {...}

    public void methodB() {
        this.methodA();  // ❌ 事务失效，因为走的是 this，不是代理
    }
}
```

- **本质**：Spring 事务通过代理生效，`this` 绕过了代理。
- **解决方案**：从 Spring 容器中获取自己的代理对象（`AopContext.currentProxy()`）或注入自己。

4. **非 public 方法上的事务**

```java
@Transactional
private void xxx() {}  // ❌ 事务失效
```

- **原因**：Spring 事务代理默认只对 `public` 方法生效（JDK 动态代理限制，CGLIB 虽能代理非 public 但框架默认不处理）。
- **建议**：只标注在 `public` 方法上。

Spring事务是通过代理对象来控制的，只有通过代理对象的方法调用才会应用事务管理的相关规则。