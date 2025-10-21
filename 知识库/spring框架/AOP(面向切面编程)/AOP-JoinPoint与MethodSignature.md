
在 [[知识库/spring框架/AOP(面向切面编程)/AOP-通知(Advice)|AOP-通知(Advice)]] 方法中，我们经常需要获取==当前被拦截方法==（即[[知识库/spring框架/AOP(面向切面编程)/AOP-核心概念(Aspect,Advice,Pointcut)|连接点 (JoinPoint)]]）的上下文信息，例如方法名、参数、注解等。`org.aspectj.lang.JoinPoint` 及其子接口 `ProceedingJoinPoint` (仅用于 `@Around` 通知) 提供了这些能力。

## 1. `JoinPoint` 接口

`JoinPoint` 是所有通知方法（除 `@Around` 外）可以接收的参数类型。它提供了访问连接点静态信息（如方法签名）和运行时信息（如方法参数）的方法。

> [!EXAMPLE] `JoinPoint` 常用方法
> ```java
> @Before("pointcut()")
> public void before(JoinPoint joinPoint) {
>     // 1. 获取目标方法的签名信息
>     Signature signature = joinPoint.getSignature();
>     System.out.println("方法签名: " + signature);
>     System.out.println("方法名: " + signature.getName());
>     System.out.println("声明类型: " + signature.getDeclaringTypeName()); // 方法所在类的全名
>
>     // 2. 获取传递给目标方法的参数值 (数组)
>     Object[] args = joinPoint.getArgs();
>     System.out.println("方法参数: " + java.util.Arrays.toString(args));
>
>     // 3. 获取目标对象 (被代理的原始对象)
>     Object target = joinPoint.getTarget();
>     System.out.println("目标对象: " + target);
>
>     // 4. 获取代理对象本身
>     Object proxy = joinPoint.getThis();
>     System.out.println("代理对象: " + proxy);
> }
> ```

## 2. `Signature` 与 `MethodSignature`

`joinPoint.getSignature()` 返回的是一个 `org.aspectj.lang.Signature` 接口。这是一个比较通用的接口，因为它需要兼容 AspectJ 可能支持的其他连接点类型。

然而，在 Spring AOP 中，连接点==总是方法的执行==。因此，`Signature` 对象实际上**总是** `org.aspectj.lang.reflect.MethodSignature` 接口的一个实例。

> [!NOTE] `MethodSignature` 的优势
> `MethodSignature` 接口继承自 `Signature`，并提供了==更具体的方法相关信息==，最重要的是它提供了获取==`java.lang.reflect.Method` 对象==的能力。
>
> ```java
> if (joinPoint.getSignature() instanceof MethodSignature) {
>     MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
>
>     // 获取返回类型
>     Class<?> returnType = methodSignature.getReturnType();
>
>     // 获取方法参数类型 (Class<?> 数组)
>     Class<?>[] parameterTypes = methodSignature.getParameterTypes();
>
>     // 获取方法参数名称 (String[] 数组 - 需要编译时支持)
>     String[] parameterNames = methodSignature.getParameterNames();
>
>     // == 获取核心的 Method 对象 ==
>     Method method = methodSignature.getMethod();
> }
> ```
> * **==强制类型转换==**：因为我们确定 Spring AOP 中一定是 `MethodSignature`，所以可以直接进行 `(MethodSignature) joinPoint.getSignature()` 的强制类型转换。
> * **==获取 `Method` 对象==**：`methodSignature.getMethod()` 是**关键**，它返回了 [[知识库/java基础/Java-反射/Java-反射-Method对象|Java-反射-Method对象]]，使我们能够进一步通过反射获取注解、调用方法等。

## 3. `ProceedingJoinPoint` (用于 `@Around` 通知)

`ProceedingJoinPoint` 是 `JoinPoint` 的子接口，==仅用于 `@Around` (环绕通知)==。

> [!NOTE] `ProceedingJoinPoint` 的特有功能
> 除了继承 `JoinPoint` 的所有方法外，`ProceedingJoinPoint` 最重要的方法是：
> * **`Object proceed()`**: ==执行目标方法==。
> * **`Object proceed(Object[] args)`**: 使用==修改后的参数==执行目标方法。
>
> 这使得 `@Around` 通知能够控制目标方法的执行时机、次数，甚至修改参数和返回值。

## 4. 在项目中的应用

在 `AutoFillAspect` 的 `before` 通知中，我们大量使用了 `JoinPoint` 和 `MethodSignature`。

> [!NOTE] 应用场景分析
> 1.  **获取方法签名**：`MethodSignature signature = (MethodSignature) joinPoint.getSignature();` 用于获取被拦截方法的详细签名信息。
> 2.  **获取 `Method` 对象**：`signature.getMethod()` 是获取方法上 `@AutoFill` 注解的前提。[[项目实践/苍穹外卖/公共字段自动填充/具体实现/4.实现AOP通知(before)|4.实现AOP通知(before)]]
> 3.  **获取方法参数**：`Object[] args = joinPoint.getArgs();` 用于获取传递给 Mapper 方法的实体对象 `entity = args[0]`，这是进行反射赋值的目标。[[项目实践/苍穹外卖/公共字段自动填充/具体实现/5.反射调用|5.反射调用]]

---
**返回MOC：**
[[知识库/spring框架/AOP(面向切面编程)/AOP (MOC)|AOP (MOC)]]