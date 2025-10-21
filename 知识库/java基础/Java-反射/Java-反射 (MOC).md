
Java 反射 (Reflection) 是 Java 语言提供的一种==强大的自省 (Introspection) 机制==。它允许程序在==运行时==获取任何一个已知名称的 `Class` 的内部信息（包括其修饰符、字段、方法、构造函数等），并且可以在运行时==动态地创建对象、调用方法、修改字段==，即使这些成员是私有的。

> [!NOTE] 核心概念
> 反射的核心在于 `java.lang.Class` 类以及 `java.lang.reflect` 包中的 `Method`, `Field`, `Constructor` 等类。通过这些类，我们可以像“照镜子”一样反观和操作 Java 对象和类。

> [!WARNING] 反射的优缺点
> * **优点**：==灵活性高==，是许多框架（如 Spring, MyBatis）实现依赖注入 (DI)、AOP、动态代理、注解处理等功能的基石。
> * **缺点**：
>     * ==性能开销大==：反射操作通常比直接代码调用慢得多。
>     * ==破坏封装性==：可以绕过访问修饰符（如 `private`）直接访问和修改成员。
>     * ==类型不安全==：编译器无法检查反射调用的类型错误，错误通常在运行时才会暴露。

## 核心知识点
- [[知识库/java基础/Java-反射/Java-反射-Class对象|Java-反射-Class对象]]：反射的入口，代表一个类的“结构图”。
- [[知识库/java基础/Java-反射/Java-反射-Method对象|Java-反射-Method对象]]：代表类中的方法，可以动态调用。
- [[知识库/java基础/Java-反射/Java-反射-Field对象|Java-反射-Field对象]]：代表类中的字段，可以动态读写。

## 在项目中的应用
反射是实现“公共字段自动填充”动态赋值的关键技术。
- [[项目实践/苍穹外卖/公共字段自动填充/具体实现/5.反射调用|5.反射调用]]：在 `AutoFillAspect` 中，我们使用 `entity.getClass().getDeclaredMethod()` 获取 `setter` 方法对应的 `Method` 对象，然后使用 `method.invoke(entity, ...)` 动态调用这些方法，为实体对象赋值。
- [[项目实践/苍穹外卖/公共字段自动填充/具体实现/4.实现AOP通知(before)|4.实现AOP通知(before)]]：通过 `methodSignature.getMethod().getAnnotation(...)` 读取方法上的 `@AutoFill` 注解。