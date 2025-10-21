`enum` (枚举) 是 Java 提供的一种特殊的**==引用数据类型==**，它允许我们定义一个类型，这个类型的变量==只能取一组预先定义好的常量值==。

## 1. 为什么需要枚举？

在没有枚举之前，我们通常使用==常量==（`public static final int`）或者==字符串==来表示一组固定的选项（例如：星期一到星期天，订单状态）。

> [!WARNING] 使用常量或字符串的问题
> * **类型不安全**：如果用 `int` 表示星期，`1` 代表星期一，那么可以错误地传入 `8` 甚至 `-1`，编译器无法检查。如果用 `String`，则 `"MONDAY"` 和 `"Monday"` 是不同的值。
> * **可读性差**：`if (day == 1)` 不如 `if (day == DayOfWeek.MONDAY)` 清晰。
> * **维护困难**：增删选项时可能需要在多处修改常量值或字符串比较逻辑。

枚举完美地解决了这些问题。

## 2. 基本语法

定义一个枚举非常简单：

> [!EXAMPLE] 语法示例
> ```java
> public enum DayOfWeek {
>     MONDAY,
>     TUESDAY,
>     WEDNESDAY,
>     THURSDAY,
>     FRIDAY,
>     SATURDAY,
>     SUNDAY
> }
> ```
> * 使用 `enum` 关键字代替 `class` 或 `interface`。
> * 大括号内列出所有可能的常量值（实例），通常用==大写字母==表示，用逗号分隔。
> * 每个枚举常量（如 `MONDAY`）都是 `DayOfWeek` 类型的==一个静态、最终的实例==。

## 3. 使用枚举

> [!EXAMPLE] 使用示例
> ```java
> // 声明变量
> DayOfWeek today;
>
> // 赋值 (只能赋预定义的常量)
> today = DayOfWeek.MONDAY;
>
> // 比较
> if (today == DayOfWeek.MONDAY) {
>     System.out.println("开始新的一周！");
> }
>
> // switch 语句 (非常常用)
> switch (today) {
>     case MONDAY:
>         System.out.println("星期一");
>         break;
>     case TUESDAY:
>         System.out.println("星期二");
>         break;
>     // ...
>     default:
>         System.out.println("周末");
> }
> ```

## 4. 在项目中的应用

在“苍穹外卖”项目中，我们定义了 `OperationType` 枚举来表示数据库的操作类型。

> [!EXAMPLE] OperationType.java (Code)
> ```java
> package com.sky.enumeration;
>
> /**
>  * 数据库操作类型
>  */
> public enum OperationType {
>
>     /**
>      * 更新操作
>      */
>     UPDATE,
>
>     /**
>      * 插入操作
>      */
>     INSERT
>
> }
> ```
>

> [!NOTE] 应用场景
> 1.  **作为注解属性类型**：在 [[项目实践/苍穹外卖/公共字段自动填充/具体实现/1.定义注解@AutoFill|1.定义注解@AutoFill]] 中，`@AutoFill` 注解的 `value` 属性被声明为 `OperationType value();`，强制使用者必须传入 `OperationType.INSERT` 或 `OperationType.UPDATE`。
> 2.  **逻辑判断依据**：在 [[项目实践/苍穹外卖/公共字段自动填充/具体实现/4.实现AOP通知(before)|4.实现AOP通知(before)]] 中，`AutoFillAspect` 通过读取注解的 `value` 值，使用 `if (operationType == OperationType.INSERT)` 来区分不同的操作，执行不同的填充逻辑。

> [!TIP] 关键优势总结
> * **==类型安全==**：防止传入无效的操作类型。
> * **==代码清晰==**：`OperationType.INSERT` 比魔法数字 `1` 或字符串 `"insert"` 更易读。
> * **==易于维护==**：代码意图明确。

---
**相关链接：**
[[项目实践/苍穹外卖/公共字段自动填充/具体实现/2.定义枚举类OperationType|2.定义枚举类OperationType]]