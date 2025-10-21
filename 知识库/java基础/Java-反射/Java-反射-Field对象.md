
`java.lang.reflect.Field` 对象是 Java 反射机制中用来==代表一个类或接口中的特定字段（成员变量）==的类。

> [!NOTE] 核心概念
> * **==字段的“代理”==**：你可以把 `Field` 对象看作是某个具体字段（例如 `Employee` 类 的 `private String name;` 字段）在运行时的==“代理”或“句柄”==。
> * **==包含字段信息==**：它封装了关于这个字段的所有信息，包括字段名、数据类型、修饰符（`public`, `private`, `static`, `final` 等）以及字段上的注解。
> * **==核心能力：动态读写==**：`Field` 对象最强大的能力是允许你在运行时==动态地读取或修改==它所代表的字段的值，即使这个字段是 `private` 的。

## 1. 获取 Field 对象

通常通过 [[知识库/java基础/Java-反射/Java-反射-Class对象|Java-反射-Class对象]] 来获取 `Field` 对象。

> [!EXAMPLE] 获取 Field 对象的方法
> ```java
> Class<?> employeeClass = Employee.class;
>
> // 1. getDeclaredField(String name)
> // 获取本类声明的指定名称的字段（包括 private），但不包括继承的字段
> try {
>     Field nameField = employeeClass.getDeclaredField("name"); // 获取名为 "name" 的字段
>     Field idField = employeeClass.getDeclaredField("id");     // 获取名为 "id" 的字段
> } catch (NoSuchFieldException e) {
>     // 如果字段未找到，会抛出此异常
>     e.printStackTrace();
> }
>
> // 2. getField(String name)
> // 获取指定的 public 字段，包括从父类或接口继承来的 public 字段
> try {
>     // 假设 Employee 有一个 public static final String COMPANY = "Sky";
>     // Field companyField = employeeClass.getField("COMPANY"); // 获取 public 字段
> } catch (NoSuchFieldException e) {
>     e.printStackTrace();
> }
>
> // 3. getDeclaredFields() / getFields()
> // 获取所有声明的字段 / 所有 public 字段 (返回 Field[] 数组)
> Field[] allDeclaredFields = employeeClass.getDeclaredFields();
> Field[] allPublicFields = employeeClass.getFields();
> ```

> [!TIP] `getDeclaredField` vs `getField`
> * `==getDeclaredField==`：查找范围仅限==本类==，但可以找到==所有访问级别==（public, protected, default, private）的字段。
> * `==getField==`：查找范围包括==本类及父类/接口==，但==只能==找到 `==public==` 字段。
> * 在需要访问特定类的（包括非 public）字段时，通常使用 `getDeclaredField`。

## 2. 核心用法：`get()` 读取 与 `set()` 修改

`Field` 对象的核心方法是 `get(Object obj)` 和 `set(Object obj, Object value)`。

> [!EXAMPLE] 使用 get() 和 set()
> ```java
> Employee employee = new Employee();
> employee.setName("初始名"); // 假设 name 是 private
>
> try {
>     Field nameField = Employee.class.getDeclaredField("name");
>
>     // --- 读取字段值 ---
>     // 关键：如果字段是 private，必须先设置为可访问
>     nameField.setAccessible(true);
>
>     // get(obj): 获取指定对象 obj 上的该字段的值
>     Object nameValue = nameField.get(employee);
>     System.out.println(nameValue); // 输出 "初始名"
>
>     // --- 修改字段值 ---
>     // set(obj, value): 将指定对象 obj 上的该字段的值修改为 value
>     nameField.set(employee, "修改后的名");
>
>     // 再次读取验证
>     nameValue = nameField.get(employee);
>     System.out.println(nameValue); // 输出 "修改后的名"
>
>     // 如果操作的是静态字段 (static)，obj 参数传入 null
>     // Field staticField = ...
>     // staticField.setAccessible(true);
>     // Object staticValue = staticField.get(null);
>     // staticField.set(null, newValue);
>
> } catch (NoSuchFieldException | IllegalAccessException e) {
>     // IllegalAccessException: 如果字段不可访问 (例如 private 且未调用 setAccessible(true))
>     e.printStackTrace();
> }
> ```

> [!NOTE] `setAccessible(true)`
> 这是反射中==“暴力破解”封装性==的关键方法。默认情况下，你无法通过反射访问 `private` 成员。调用 `setAccessible(true)` 会**==取消 Java 语言访问检查==**，允许你读取和修改 `private` 字段或调用 `private` 方法。
> **==必须谨慎使用==**，因为它破坏了类的封装设计。

## 3. 其他常用方法 (已修正格式)

* `getName()`: 获取字段名 (`String`)。
* `getType()`: 获取字段的数据类型 (`Class<?>`)。
* `getModifiers()`: 获取字段的修饰符 (`int`，需要使用 `Modifier` 类解析)。
* `getAnnotation(Class<A> annotationClass)`: 获取字段上的指定类型的注解。
* `getDeclaredAnnotations()`: 获取字段上声明的所有注解。

## 4. 在项目中的应用

在“苍穹外卖”项目的“公共字段自动填充”功能中，**==我们没有直接使用 `Field` 对象来修改字段值==**。

> [!NOTE] 应用场景分析 (对比)
> * **当前方案**：我们选择了通过 [[知识库/java基础/Java-反射/Java-反射-Method对象|Java-反射-Method对象]] 的 `invoke()` 方法来调用实体的 `setter` 方法 (如 `setCreateTime()`) 来完成赋值。[[项目实践/苍穹外卖/公共字段自动填充/具体实现/5.反射调用|5.反射调用]]
> * **`Field.set()` 方案 (未使用)**：理论上，我们也可以通过 `Field.set()` 直接修改 `createTime` 等字段的值，即使它们是 `private` 的。
> * **==选择 `Method.invoke()` 的原因==**：
>     * ==更符合封装原则==：调用 `setter` 方法是标准的 Java Bean 操作方式，`setter` 方法内部可能包含一些校验或其他逻辑（虽然本项目中的实体类很简单，没有这些逻辑）。直接使用 `Field.set()` 会绕过这些潜在逻辑。
>     * ==代码意图更清晰==：调用 `setCreateTime` 比直接设置 `createTime` 字段更能表达“设置创建时间”的意图。
>     * ==避免 `setAccessible(true)`==：`setter` 方法通常是 `public` 的，调用 `invoke()` 不需要“暴力破解”访问权限，代码更简洁、风险更小。

---
**返回MOC：**
[[知识库/java基础/Java-反射/Java-反射 (MOC)|Java-反射 (MOC)]]