
[[知识库/spring框架/AOP(面向切面编程)/AOP (MOC)|AOP (MOC)]] 旨在将==横切关注点==从业务逻辑中分离出来。为了理解 AOP 如何工作，需要掌握以下核心概念：

## 1. 连接点 (JoinPoint)

> [!NOTE] 定义
> **连接点 (JoinPoint)** 是指在程序执行过程中==可以插入切面==的特定点。
>
> * 在 Spring AOP 中，连接点==总是方法的执行==。例如，调用 `EmployeeMapper.insert()` 方法就是一个连接点。
> * 其他 AOP 实现（如 AspectJ）可能支持更广泛的连接点，如字段访问、构造函数调用等，但 Spring AOP 不支持。
> * 连接点是一个==运行时==的概念。

## 2. 切点 (Pointcut)

> [!NOTE] 定义
> **切点 (Pointcut)** 是一个==谓词或表达式==，用于==匹配==一组连接点 (JoinPoint)。
>
> * ==作用==：切点定义了**“在哪些连接点上应用通知 (Advice)”**。
> * ==实现方式==：通常使用特定的==切点表达式语言==来定义，例如 Spring AOP 支持的 AspectJ 切点表达式。[[知识库/spring框架/AOP(面向切面编程)/AOP-切点表达式(Pointcut)|AOP-切点表达式(Pointcut)]]
> * ==例子==：`execution (* com.sky.mapper.*.*(..))` 就是一个切点表达式，它匹配 `com.sky.mapper` 包下所有类的所有方法的执行（连接点）。

## 3. 通知 (Advice)

> [!NOTE] 定义
> **通知 (Advice)** 是切面在==特定连接点==上执行的==实际操作==或==代码逻辑==。
>
> * ==作用==：定义了切面**“做什么”**以及**“什么时候做”**。
> * ==类型==：Spring AOP 支持多种类型的通知，通过注解来声明。[[知识库/spring框架/AOP(面向切面编程)/AOP-通知(Advice)|AOP-通知(Advice)]]
>     * `@Before`：前置通知，在连接点==之前==执行。
>     * `@AfterReturning`：返回通知，在连接点正常==返回后==执行。
>     * `@AfterThrowing`：异常通知，在连接点抛出==异常后==执行。
>     * `@After`：后置通知，无论连接点是正常返回还是抛出异常，==之后都会==执行。
>     * `@Around`：环绕通知，==包围==连接点的执行，是最强大的通知类型，可以自定义执行时机和流程。
> * ==例子==：`AutoFillAspect` 中的 `before(JoinPoint joinPoint)` 方法就是一个 `@Before` 通知。[[项目实践/苍穹外卖/公共字段自动填充/具体实现/4.实现AOP通知(before)|4.实现AOP通知(before)]]

## 4. 切面 (Aspect)

> [!NOTE] 定义
> **切面 (Aspect)** 是==通知 (Advice)== 和==切点 (Pointcut)== 的==结合==。它定义了哪个通知应该在哪个切点上执行。
>
> * ==作用==：将横切关注点模块化。一个切面封装了一个特定的横切功能（如日志、事务、自动填充）。
> * ==实现方式==：在 Spring AOP 中，通常是一个带有 `@Aspect` 注解的 Java 类。
> * ==例子==：`AutoFillAspect` 类就是一个切面，它包含了 `pointcut()` 定义的切点和 `before()` 定义的通知。

## 5. 织入 (Weaving)

> [!NOTE] 定义
> **织入 (Weaving)** 是指将==切面 (Aspect)== 应用到==目标对象 (Target Object)==上，从而创建==代理对象 (Proxy Object)==的过程。
>
> * ==作用==：在目标方法的连接点上动态地插入通知代码。
> * ==时机==：织入可以在编译时、类加载时或运行时完成。
>     * **Spring AOP** 主要在==运行时==进行织入，通过==动态代理==（JDK Dynamic Proxy 或 CGLIB）技术创建代理对象。当调用代理对象的方法时，代理对象会根据切面定义，在调用目标对象方法之前、之后或环绕执行通知逻辑。

## 6. 目标对象 (Target Object)

> [!NOTE] 定义
> **目标对象 (Target Object)** 是指==被一个或多个切面通知==的对象。也就是包含连接点的对象，即你的原始业务逻辑类（如 `EmployeeServiceImpl` 或 `EmployeeMapper` 接口的实现类）。

## 7. AOP 代理 (AOP Proxy)

> [!NOTE] 定义
> **AOP 代理 (AOP Proxy)** 是 AOP 框架（如 Spring AOP）创建的对象，用于==实现切面契约==。它包含了目标对象的逻辑，并且==增加了通知的逻辑==。
>
> * 在 Spring AOP 中，代理是 JDK 动态代理或 CGLIB 代理。应用程序**直接与代理对象交互**，而不是目标对象。

---
**返回MOC：**
[[知识库/spring框架/AOP(面向切面编程)/AOP (MOC)|AOP (MOC)]]