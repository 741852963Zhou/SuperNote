# AOP (MOC)

==AOP (Aspect-Oriented Programming)==，即**面向切面编程**，是一种编程范式，旨在通过==分离横切关注点 (Cross-Cutting Concerns)==来提高软件系统的模块化程度。

> [!NOTE] 核心思想
> 在软件开发中，除了核心业务逻辑（如用户管理、订单处理），还存在大量分散在各个模块中的==通用功能==，例如：
> * 日志记录
> * 事务管理
> * 权限控制
> * 性能监控
> * **公共字段填充** (我们项目中的例子)
>
> 这些通用功能被称为==横切关注点==，因为它们“横切”了多个核心业务模块。如果将这些功能的代码直接硬编码到每个业务模块中，会导致代码重复、耦合度高、难以维护。
>
> AOP 的目标就是将这些横切关注点==从核心业务逻辑中剥离出来==，封装成独立的“切面 (Aspect)”，然后在需要的时候==动态地==将这些切面“织入 (Weave)”到业务逻辑中，而==无需修改业务逻辑本身==。

## 核心知识点

* [[知识库/spring框架/AOP(面向切面编程)/AOP-核心概念(Aspect,Advice,Pointcut)|AOP-核心概念(Aspect,Advice,Pointcut)]]：理解切面 (Aspect)、通知 (Advice)、切点 (Pointcut)、连接点 (JoinPoint)、织入 (Weaving) 等基本术语。
* [[知识库/spring框架/AOP(面向切面编程)/AOP-切点表达式(Pointcut)|AOP-切点表达式(Pointcut)]]：学习如何使用表达式（如 `execution`, `@annotation`）来精确定义拦截的目标方法。
* [[知识库/spring框架/AOP(面向切面编程)/AOP-通知(Advice)|AOP-通知(Advice)]]：了解不同类型的通知（如 `@Before`, `@After`, `@Around`）以及它们的执行时机。
* [[知识库/spring框架/AOP(面向切面编程)/AOP-JoinPoint与MethodSignature|AOP-JoinPoint与MethodSignature]]：掌握如何在通知方法中获取被拦截方法的信息。

## 在项目中的应用

在“苍穹外卖”项目中，AOP 主要用于实现**==公共字段自动填充==**功能。

> [!NOTE] 应用场景分析
> * **定义切面**：我们创建了 `AutoFillAspect` 类，并使用 `@Aspect` 注解将其声明为一个切面。
> * **定义切点**：使用 `@Pointcut` 注解定义了切点表达式 `"execution (* com.sky.mapper.*.*(..)) && @annotation(com.sky.annotation.AutoFill)"`，精确拦截 `com.sky.mapper` 包下所有带有 `@AutoFill` 注解的方法。[[项目实践/苍穹外卖/公共字段自动填充/具体实现/3.定义AOP切面(Pointcut)|3.定义AOP切面(Pointcut)]]
> * **定义通知**：使用 `@Before` 注解定义了前置通知 `before(JoinPoint joinPoint)`，在目标方法执行前执行字段填充逻辑。[[项目实践/苍穹外卖/公共字段自动填充/具体实现/4.实现AOP通知(before)|4.实现AOP通知(before)]]
> * **获取信息与执行**：通知方法通过 `JoinPoint` 获取方法签名和参数，通过 [[知识库/java基础/Java-反射/Java-反射 (MOC)|Java-反射 (MOC)]] 读取注解信息并动态调用 `setter` 方法完成填充。[[项目实践/苍穹外卖/公共字段自动填充/具体实现/5.反射调用|5.反射调用]]

---
**相关链接：**
[[项目实践/苍穹外卖/公共字段自动填充/解决方案|解决方案]]