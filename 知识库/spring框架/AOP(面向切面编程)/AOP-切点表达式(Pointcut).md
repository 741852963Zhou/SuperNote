
[[知识库/spring框架/AOP(面向切面编程)/AOP-核心概念(Aspect,Advice,Pointcut)|AOP-核心概念(Aspect,Advice,Pointcut)]] 中的==切点 (Pointcut)== 定义了==“在哪里应用通知 (Advice)”==。Spring AOP 使用**AspectJ 的切点表达式语言**来精确地指定要拦截的==连接点 (JoinPoint)==（在 Spring AOP 中即方法的执行）。

## 1. 主要的切点指示器 (Pointcut Designators - PCD)

Spring AOP 支持多种 AspectJ 的切点指示器，最常用的是 `execution` 和 `@annotation`。

### 1.1. `execution`

这是==最核心、最常用==的指示器，用于匹配==方法执行==的连接点。

> [!EXAMPLE] `execution` 语法结构
> ```
> execution(<修饰符模式>? <返回类型模式> <声明类型模式>? <方法名模式>(<参数模式>) <异常模式>?)
> ```
> * `?` 表示可选部分。
> * **`<修饰符模式>`**: 方法的可见性，如 `public`, `protected` 等。（通常省略，匹配所有）
> * **`<返回类型模式>`**: 方法的返回类型。`*` 匹配任意返回类型。
> * **`<声明类型模式>`**: 方法所在的类的全限定名。可以使用 `*` 和 `..` 通配符。
>     * `*`: 匹配任意字符序列（不包括 `.`)。
>     * `..`: 匹配任意字符序列（包括 `.`)，通常用于匹配包及其子包。
> * **`<方法名模式>`**: 要匹配的方法名。`*` 匹配任意方法名。
> * **`<参数模式>`**: 方法的参数列表。
>     * `()`: 匹配无参方法。
>     * `(..)`: 匹配任意数量、任意类型的参数。
>     * `(*)`: 匹配只有一个任意类型参数的方法。
>     * `(String, *)`: 匹配第一个参数是 String，第二个参数是任意类型的方法。
> * **`<异常模式>`**: 方法抛出的异常类型。（通常省略）

> [!TIP] 示例解析
> 在 `AutoFillAspect` 中使用的 `execution(* com.sky.mapper.*.*(..))`：
> * `*`: 匹配任意返回类型。
> * `com.sky.mapper`: 指定包名。
> * `.*`: 匹配该包下的任意类 (`.` 后面的 `*`)。
> * `.*`: 匹配类中的任意方法 (`.` 后面的 `*`)。
> * `(..)`: 匹配任意参数。
> * **含义**：匹配 `com.sky.mapper` 包下所有类的所有方法的执行。

### 1.2. `@annotation`

用于匹配==持有指定注解==的连接点（即方法）。

> [!EXAMPLE] `@annotation` 语法
> ```
> @annotation(<注解类型的全限定名>)
> ```
> * **`<注解类型的全限定名>`**: 需要匹配的注解的完整类名。

> [!TIP] 示例解析
> 在 `AutoFillAspect` 中使用的 `@annotation(com.sky.annotation.AutoFill)`：
> * 匹配所有被 `com.sky.annotation.AutoFill` 这个 [[知识库/java基础/Java-注解(Annotation)/Java-注解-自定义注解|Java-注解-自定义注解]] 所标记的方法。

### 1.3. 其他常用指示器 (了解)

* **`within(<类型模式>)`**: 匹配指定类型内的所有连接点（方法执行）。例如 `within(com.sky.service.*)` 匹配 `service` 包下所有类的方法。
* **`args(<类型列表>)`**: 匹配当前执行的方法传入的参数是指定类型的方法。例如 `args(String, ..)` 匹配第一个参数是 String 的方法。
* **`@within(<注解类型>)`**: 匹配持有指定注解的==类型==内的所有方法。
* **`@args(<注解类型列表>)`**: 匹配传入的==实际参数对象==的运行时类型持有指定注解的方法。
* **`bean(<bean名称或通配符>)`**: 匹配特定名称的 Spring Bean 上的方法。

## 2. 组合切点表达式

可以使用逻辑运算符 `&&` (与), `||` (或), `!` (非) 来组合多个切点表达式。

> [!EXAMPLE] 组合示例
> ```java
> // 同时满足 execution 和 @annotation 条件
> @Pointcut("execution (* com.sky.mapper.*.*(..)) && @annotation(com.sky.annotation.AutoFill)")
> public void pointcut() {}
> ```
>
> * `&&`: 表示必须同时匹配 `com.sky.mapper` 包下的方法 **并且** 该方法还带有 `@AutoFill` 注解。

## 3. 在项目中的应用

`AutoFillAspect` 使用了 `execution` 和 `@annotation` 的组合，以精确地定位到需要进行自动填充的 Mapper 方法。

> [!NOTE] 应用场景分析
> * **`execution` 划定范围**：`execution(* com.sky.mapper.*.*(..))` 首先将拦截范围限定在 `mapper` 包下的所有方法，这是一个比较粗粒度的匹配。
> * **`@annotation` 精确筛选**：`@annotation(com.sky.annotation.AutoFill)` 则进一步筛选出那些**真正需要**自动填充逻辑的方法（即被 `@AutoFill` 注解标记的方法）。
> * **`&&` 组合**：确保只有同时满足这两个条件的方法才会被拦截，既保证了功能的正确性，也避免了不必要的性能开销。

---
**返回MOC：**
[[知识库/spring框架/AOP(面向切面编程)/AOP (MOC)|AOP (MOC)]]