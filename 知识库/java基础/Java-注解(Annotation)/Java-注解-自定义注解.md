
除了使用 Java 内置的注解（如 `@Override`, `@Deprecated`）和第三方库提供的注解（如 Spring 的 `@Component`, `@Autowired`），我们还可以==创建自己的注解==来满足特定需求。

## 1. 定义语法

自定义注解使用 `@interface` 关键字来定义，它本质上是一种特殊的==接口==。

> [!EXAMPLE] 基本语法
> ```java
> import java.lang.annotation.*;
>
> // 使用元注解来修饰自定义注解
> @Retention(RetentionPolicy.RUNTIME) // 指定注解的生命周期
> @Target(ElementType.METHOD)       // 指定注解可以应用的目标
>
> public @interface MyCustomAnnotation {
>     // 在这里定义注解的属性
> }
> ```
> * `@interface` 关键字用于声明一个注解。
> * 通常需要使用 [[知识库/java基础/Java-注解(Annotation)/Java-注解-元注解|Java-注解-元注解]] (`@Retention`, `@Target` 等) 来指定自定义注解的行为。

## 2. 定义属性

注解可以包含“属性”（也叫“元素”或“成员”），用于在使用注解时传递额外的信息。

> [!NOTE] 属性定义的特殊语法
> 注解的属性被声明为==无参数的方法==。
> ```java
> public @interface MyAnnotationWithAttributes {
>     // 声明一个名为 description 的 String 类型属性
>     String description();
>
>     // 声明一个名为 count 的 int 类型属性，并提供默认值
>     int count() default 1;
>
>     // 声明一个名为 tags 的 String 数组类型属性
>     String[] tags() default {};
> }
> ```
> * 方法名 (`description`, `count`, `tags`) 就是属性名。
> * 方法的==返回类型==就是属性的数据类型。允许的类型包括：
>     * 基本数据类型 (`int`, `float`, `boolean`, etc.)
>     * `String`
>     * `Class` (e.g., `Class<?> entityClass();`)
>     * 枚举类型 (`enum`)
>     * 另一个注解类型
>     * 以上所有类型的数组 (e.g., `String[]`)
> * 可以使用 `default` 关键字为属性指定==默认值==。如果在使用注解时没有显式提供该属性的值，就会使用默认值。

> [!TIP] `value` 属性的特殊约定
> 如果一个注解**只有一个属性**，并且该属性的名字是 `value`，那么在使用该注解时，可以省略属性名 `value =`，直接写值。
> ```java
> public @interface SingleValueAnnotation {
>     String value(); // 属性名必须是 value
> }
>
> // 使用时可以简写
> @SingleValueAnnotation("some value")
> // 等价于
> @SingleValueAnnotation(value = "some value")
> ```
> 如果注解有多个属性，或者唯一的属性名不是 `value`，则必须使用 `属性名 = 值` 的形式。

## 3. 在项目中的应用

在“苍穹外卖”项目中，我们自定义了 `@AutoFill` 注解，用于标记需要进行公共字段自动填充的 Mapper 方法。

> [!EXAMPLE] @AutoFill.java (Code)
> ```java
> package com.sky.annotation;
>
> import com.sky.enumeration.OperationType; // 引入枚举类型
> import java.lang.annotation.*;
>
> /**
>  * 自定义注解，用于标识某个方法需要进行功能字段自动填充处理
>  */
> @Target(ElementType.METHOD)           // 只能用于方法
> @Retention(RetentionPolicy.RUNTIME)   // 运行时可见 (AOP需要)
> public @interface AutoFill {
>     // 定义一个名为 value 的属性，类型为 OperationType 枚举
>     OperationType value();
> }
> ```
>

> [!NOTE] 应用场景分析
> * **定义**：使用了 `@Target` 和 `@Retention` 两个 [[知识库/java基础/Java-注解(Annotation)/Java-注解-元注解|Java-注解-元注解]]。`@Retention(RetentionPolicy.RUNTIME)` 确保 AOP 切面能在运行时通过反射读取到此注解。
> * **属性**：定义了==唯一的属性 `value`==，其类型是 [[知识库/java基础/Java-枚举(Enum)|Java-枚举(Enum)]] `OperationType`。这使得我们可以通过 `value` 属性来区分 `INSERT` 和 `UPDATE` 操作。
> * **使用**：由于属性名为 `value`，我们在 Mapper 方法上可以简洁地使用 `@AutoFill(OperationType.INSERT)`。
> * **读取**：在 [[项目实践/苍穹外卖/公共字段自动填充/具体实现/4.实现AOP通知(before)|4.实现AOP通知(before)]] 中，`AutoFillAspect` 通过反射的 `getAnnotation(AutoFill.class)` 获取注解实例，然后调用 `autoFill.value()` 来获取实际的操作类型。

---
**返回MOC：**
[[知识库/java基础/Java-注解(Annotation)/Java-注解 (MOC)|Java-注解 (MOC)]]