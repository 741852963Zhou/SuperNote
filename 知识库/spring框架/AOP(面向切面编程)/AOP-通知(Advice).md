
[[知識庫/spring框架/AOP(面向切面编程)/AOP-核心概念(Aspect,Advice,Pointcut)|AOP-核心概念(Aspect,Advice,Pointcut)]] 中的==通知 (Advice)== 定义了切面**“做什么”**以及**“什么时候做”**。它是切面在匹配的[[知識庫/spring框架/AOP(面向切面编程)/AOP-核心概念(Aspect,Advice,Pointcut)|连接点 (JoinPoint)]]上执行的==实际代码逻辑==。

Spring AOP 支持多种类型的通知，通过注解的方式与切点关联。

## 1. 通知类型

Spring AOP 主要支持以下五种通知类型：

### 1.1. `@Before` (前置通知)

> [!NOTE] 定义与时机
> * **定义**：在目标方法（连接点）==执行之前==执行的通知。
> * **特点**：无法阻止目标方法的执行（除非它抛出异常）。
> * **应用场景**：执行预处理操作，如权限检查、参数校验、==设置默认值（如此项目的公共字段填充）==。

### 1.2. `@AfterReturning` (返回通知)

> [!NOTE] 定义与时机
> * **定义**：在目标方法==正常执行并返回结果之后==执行的通知。
> * **特点**：可以访问到目标方法的==返回值==。
> * **应用场景**：对返回值进行处理、记录成功日志等。

### 1.3. `@AfterThrowing` (异常通知)

> [!NOTE] 定义与时机
> * **定义**：在目标方法执行过程中==抛出异常之后==执行的通知。
> * **特点**：可以访问到抛出的==异常对象==。
> * **应用场景**：记录异常日志、进行异常转换、执行回滚操作等。

### 1.4. `@After` (后置通知)

> [!NOTE] 定义与时机
> * **定义**：==无论目标方法是正常返回还是抛出异常==，在方法执行==之后都会==执行的通知。
> * **特点**：类似于 `finally` 块，通常用于==资源清理==。
> * **注意**：它无法访问返回值或异常对象。

### 1.5. `@Around` (环绕通知)

> [!NOTE] 定义与时机
> * **定义**：==包围==目标方法执行的通知，是最强大的一种通知。
> * **特点**：
>     * 可以在目标方法==执行前后==自定义行为。
>     * 可以==决定是否执行==目标方法。
>     * 可以==改变==目标方法的==参数==或==返回值==。
> * **实现**：环绕通知方法需要接收 `ProceedingJoinPoint` 类型的参数，并**显式调用 `proceedingJoinPoint.proceed()`** 来执行目标方法。
> * **应用场景**：事务管理、性能监控、缓存等需要完全控制方法执行流程的场景。

## 2. 通知方法的参数

通知方法可以接收一个可选的 `JoinPoint` (或其子接口 `ProceedingJoinPoint`，仅用于 `@Around`) 参数，以获取连接点的信息。

> [!TIP] 获取连接点信息
> * `joinPoint.getArgs()`: 获取方法参数。
> * `joinPoint.getSignature()`: 获取方法签名。
> * `joinPoint.getTarget()`: 获取目标对象。
> * `joinPoint.getThis()`: 获取代理对象。
>
> 详见 [[知識庫/spring框架/AOP(面向切面编程)/AOP-JoinPoint与MethodSignature|AOP-JoinPoint与MethodSignature]]。

## 3. 在项目中的应用

`AutoFillAspect` 选择了 `@Before` 通知来实现公共字段的自动填充。

> [!EXAMPLE] AutoFillAspect.java (@Before 通知)
> ```java
>     /**
>      * 前置通知，在通知中进行公共字段的赋值
>      * @param joinPoint 连接点，包含被拦截方法的信息
>      */
>     @SneakyThrows
>     @Before("pointcut()") // 引用名为 "pointcut" 的切点
>     public void before(JoinPoint joinPoint) {
>         log.info("Before AutoFillAspect...开始进行公共字段自动填充");
>         
>         // ... 获取操作类型 ...
>         // ... 获取实体对象 ...
>         // ... 准备填充数据 ...
>         // ... 通过反射执行填充 ...
>     }
> ```
>

> [!NOTE] 应用场景分析
> * **选择 `@Before` 的原因**：我们的需求是在执行 `INSERT` 或 `UPDATE` SQL 语句==之前==，就把 `createTime`, `updateUser` 等字段的值设置到实体对象中。`@Before` 通知的执行时机完美契合这个需求。
> * **参数 `JoinPoint`**：通知方法接收 `JoinPoint` 参数，以便能够获取到被拦截的 Mapper 方法（通过 `getSignature().getMethod()`）和传递给 Mapper 方法的实体对象（通过 `getArgs()[0]`）。

---
**返回MOC：**
[[知識庫/spring框架/AOP(面向切面编程)/AOP (MOC)|AOP (MOC)]]