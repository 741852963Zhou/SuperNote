# Java-反射-Class对象

`java.lang.Class` 对象是 Java 反射机制的==入口点==。它代表了 Java 虚拟机 (JVM) 中加载的一个类或接口的==运行时状态==。

> [!NOTE] 核心概念
> * **==类的“图纸”==**：你可以把 `Class` 对象想象成一个类的==“结构图”或“说明书”==。它包含了关于这个类的所有信息：类名、包名、父类、实现的接口、字段 (Fields)、方法 (Methods)、构造函数 (Constructors)、注解 (Annotations) 等。
> * **==运行时产生==**：每个 `.java` 文件编译后会生成一个 `.class` 文件。当 JVM 加载 `.class` 文件时，会在内存中创建一个对应的 `Class` 对象。==一个类在 JVM 中只对应一个 `Class` 对象==。

## 1. 获取 Class 对象的三种主要方式

在 [[项目实践/苍穹外卖/公共字段自动填充/具体实现/5.反射调用|5.反射调用]] 中，我们使用了第一种方式。

> [!EXAMPLE] 获取 Class 对象
> ```java
> // 1. 通过对象实例获取 (最常用)
> // 如果你已经有了一个对象实例，可以用 getClass() 方法
> Employee employee = new Employee();
> Class<?> classFromObject = employee.getClass(); // 返回 Employee 类的 Class 对象
>
> // 2. 通过类名获取
> // 如果你知道类的名称，可以用 .class 语法
> Class<?> classFromLiteral = Employee.class; // 返回 Employee 类的 Class 对象
>
> // 3. 通过类的完全限定名获取 (常用于框架加载配置)
> // 如果你只有一个类的字符串名称，可以用 Class.forName()
> try {
>     // 需要提供完整的包名+类名
>     Class<?> classFromString = Class.forName("com.sky.entity.Employee"); // 返回 Employee 类的 Class 对象
> } catch (ClassNotFoundException e) {
>     // 处理类未找到的异常
>     e.printStackTrace();
> }
> ```

## 2. `Class` 对象的作用

一旦获取了 `Class` 对象，你就可以通过它提供的方法来==探索和操作==这个类：

> [!EXAMPLE] 常用 Class 对象方法
> ```java
> Class<?> employeeClass = Employee.class;
>
> // 获取类名
> String className = employeeClass.getName(); // "com.sky.entity.Employee"
> String simpleName = employeeClass.getSimpleName(); // "Employee"
>
> // 获取包名
> Package pkg = employeeClass.getPackage(); // 获取 Package 对象
> String packageName = pkg.getName(); // "com.sky.entity"
>
> // 获取父类
> Class<?> superclass = employeeClass.getSuperclass(); // 获取父类的 Class 对象
>
> // 获取实现的接口
> Class<?>[] interfaces = employeeClass.getInterfaces(); // 获取所有实现的接口的 Class 对象数组
>
> // 获取字段 (Fields) - 对应 [[知识库/java基础/Java-反射/Java-反射-Field对象|Java-反射-Field对象]]
> Field[] publicFields = employeeClass.getFields(); // 获取所有 public 字段 (包括继承的)
> Field[] declaredFields = employeeClass.getDeclaredFields(); // 获取所有本类声明的字段 (不包括继承的，但包括 private)
> try {
>     Field nameField = employeeClass.getDeclaredField("name"); // 获取指定名称的字段
> } catch (NoSuchFieldException e) { e.printStackTrace(); }
>
> // 获取方法 (Methods) - 对应 [[知识库/java基础/Java-反射/Java-反射-Method对象|Java-反射-Method对象]]
> Method[] publicMethods = employeeClass.getMethods(); // 获取所有 public 方法 (包括继承的)
> Method[] declaredMethods = employeeClass.getDeclaredMethods(); // 获取所有本类声明的方法 (不包括继承的，但包括 private)
> try {
>     // 获取指定名称和参数类型的方法
>     Method setNameMethod = employeeClass.getDeclaredMethod("setName", String.class);
> } catch (NoSuchMethodException e) { e.printStackTrace(); }
>
> // 获取构造函数 (Constructors)
> Constructor<?>[] constructors = employeeClass.getDeclaredConstructors();
> try {
>     Constructor<?> defaultConstructor = employeeClass.getDeclaredConstructor(); // 获取无参构造
> } catch (NoSuchMethodException e) { e.printStackTrace(); }
>
> // 获取注解 (Annotations)
> Annotation[] annotations = employeeClass.getDeclaredAnnotations();
> boolean isData = employeeClass.isAnnotationPresent(Data.class); // 检查是否有 @Data 注解
> Data dataAnnotation = employeeClass.getAnnotation(Data.class); // 获取指定的注解实例
> ```

## 3. 在项目中的应用

在 `AutoFillAspect` 中，`Class` 对象是进行反射操作的==起点==。

> [!NOTE] 应用场景分析
> * **获取 `Class` 对象**：我们通过 `entity.getClass()` 获取被拦截方法参数 (`entity`) 的运行时 `Class` 对象。这使得我们的切面能够处理**任何类型**的实体（`Employee`, `Category` 等），只要它们符合公共字段的约定。
> * **获取 `Method` 对象**：获取到 `Class` 对象后，我们调用它的 `getDeclaredMethod()` 方法来查找具体的 `setter` 方法（如 `setCreateTime`），为后续的 [[项目实践/苍穹外卖/公共字段自动填充/具体实现/5.反射调用|5.反射调用]] 做准备。

---
**返回MOC：**
[[知识库/java基础/Java-反射/Java-反射 (MOC)|Java-反射 (MOC)]]