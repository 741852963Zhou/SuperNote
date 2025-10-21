
==元注解 (Meta-Annotations)== 是**用来注解其他注解**的注解。它们定义了自定义注解的行为。

Java 在 `java.lang.annotation` 包中内置了几种元注解，其中最常用的有 `@Retention` 和 `@Target`。

## 1. `@Retention` (生命周期)

`@Retention` 指定了被修饰的注解**可以保留到哪个阶段**。它接收一个 `RetentionPolicy` 枚举类型的参数。

> [!NOTE] `RetentionPolicy` 的三个可选值
> * **`RetentionPolicy.SOURCE`**：注解==只保留在源代码==（`.java` 文件）中，编译器编译时会**丢弃**它。通常用于做一些编译时检查，如 `@Override`, `@SuppressWarnings`。
> * **`RetentionPolicy.CLASS`**：注解会被编译器==记录在 `.class` 文件==中，但**运行时**（JVM）**无法获取**到。这是==默认值==。
> * **`RetentionPolicy.RUNTIME`**：注解会被记录在 `.class` 文件中，并且在==运行时可以通过反射获取==到。这是我们**最常用**的策略，尤其是在框架（如 Spring）和需要运行时处理注解的场景（如 AOP）中。

> [!TIP] 关键点：为什么用 RUNTIME？
> 因为我们的 [[项目实践/苍穹外卖/公共字段自动填充/具体实现/4.实现AOP通知(before)|4.实现AOP通知(before)]] 需要在**运行时**通过 [[知识库/java基础/Java-反射/Java-反射 (MOC)|Java-反射 (MOC)]] 的 `getMethod().getAnnotation(AutoFill.class)` 来读取 `@AutoFill` 注解，所以 `@AutoFill` 必须使用 `RetentionPolicy.RUNTIME` 才能在运行时可见。

## 2. `@Target` (应用目标)

`@Target` 指定了被修饰的注解**可以应用于哪些程序元素**上。它接收一个 `ElementType` 枚举类型的数组作为参数（如果只有一个目标，可以省略数组的大括号）。

> [!NOTE] `ElementType` 的常用可选值
> * **`ElementType.TYPE`**：可以应用于==类、接口（包括注解类型）、枚举==。
> * **`ElementType.FIELD`**：可以应用于==字段（成员变量）==，包括枚举常量。
> * **`ElementType.METHOD`**：可以应用于==方法==。
> * **`ElementType.PARAMETER`**：可以应用于==方法参数==。
> * **`ElementType.CONSTRUCTOR`**：可以应用于==构造函数==。
> * **`ElementType.LOCAL_VARIABLE`**：可以应用于==局部变量==。
> * **`ElementType.ANNOTATION_TYPE`**：可以应用于==注解类型==（元注解）。
> * **`ElementType.PACKAGE`**：可以应用于==包==。

> [!TIP] 关键点：为什么用 METHOD？
> 因为我们的 `@AutoFill` 注解是用来标记需要进行自动填充的 **Mapper 方法** (如 `insert`, `update`)，所以我们使用 `@Target(ElementType.METHOD)` 将其应用范围限定在方法上。

## 3. 其他元注解 (了解)

* **`@Documented`**：被此元注解修饰的注解会**包含在 Javadoc** 中。
* **`@Inherited`**：允许子类**继承**父类上的注解。默认情况下，注解是不能被继承的。
* **`@Repeatable`** (Java 8+)：允许在同一个程序元素上**重复使用**同一个注解。

## 4. 在项目中的应用

`@AutoFill` 注解 同时使用了 `@Target` 和 `@Retention` 这两个元注解。

> [!EXAMPLE] @AutoFill.java (元注解部分)
> ```java
> package com.sky.annotation;
>
> // ... 省略 import ...
>
> @Target(ElementType.METHOD)           // 只能用于方法
> @Retention(RetentionPolicy.RUNTIME)   // 运行时可见 (AOP需要)
> public @interface AutoFill {
>     OperationType value();
> }
> ```
>

> [!NOTE] 应用场景分析
> * **`@Target(ElementType.METHOD)`**：确保 `@AutoFill` 不会被误用在类、字段或其他不适合的地方。
> * **`@Retention(RetentionPolicy.RUNTIME)`**：确保 `AutoFillAspect` 能够在运行时通过反射机制读取到方法上的 `@AutoFill` 注解及其属性值。

---
**返回MOC：**
[[知识库/java基础/Java-注解(Annotation)/Java-注解 (MOC)|Java-注解 (MOC)]]