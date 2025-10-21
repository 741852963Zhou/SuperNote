
`java.lang.reflect.Method` 对象是 Java 反射机制中用来==代表一个类或接口中的特定方法==的类。

> [!NOTE] 核心概念
> * **==方法的“代理”==**：你可以把 `Method` 对象看作是某个具体方法（例如 `Employee` 类 的 `setName(String name)` 方法）在运行时的==“代理”或“句柄”==。
> * **==包含方法信息==**：它封装了关于这个方法的所有信息，包括方法名、返回类型、参数类型、修饰符（`public`, `private` 等）、抛出的异常以及方法上的注解。
> * **==核心能力：动态调用==**：`Method` 对象最强大的能力是允许你在运行时==动态地调用==它所代表的方法，即使不知道对象的具体类型。

## 1. 获取 Method 对象

通常通过 [[知识库/java基础/Java-反射/Java-反射-Class对象|Java-反射-Class对象]] 来获取 `Method` 对象。

> [!EXAMPLE] 获取 Method 对象的方法
> ```java
> Class<?> employeeClass = Employee.class;
>
> // 1. getDeclaredMethod(String name, Class<?>... parameterTypes)
> // 获取本类声明的指定方法（包括 private），但不包括继承的方法
> try {
>     // 精确匹配方法名 "setName" 和参数类型 String.class
>     Method setNameMethod = employeeClass.getDeclaredMethod("setName", String.class);
>
>     // 获取无参方法
>     Method getNameMethod = employeeClass.getDeclaredMethod("getName");
> } catch (NoSuchMethodException e) {
>     // 如果方法未找到，会抛出此异常
>     e.printStackTrace();
> }
>
> // 2. getMethod(String name, Class<?>... parameterTypes)
> // 获取指定的 public 方法，包括从父类或接口继承来的 public 方法
> try {
>     // 只能获取 public 方法
>     Method toStringMethod = employeeClass.getMethod("toString"); // 从 Object 类继承
> } catch (NoSuchMethodException e) {
>     e.printStackTrace();
> }
>
> // 3. getDeclaredMethods() / getMethods()
> // 获取所有声明的方法 / 所有 public 方法 (返回 Method[] 数组)
> Method[] allDeclaredMethods = employeeClass.getDeclaredMethods();
> Method[] allPublicMethods = employeeClass.getMethods();
> ```

> [!TIP] `getDeclaredMethod` vs `getMethod`
> * `getDeclaredMethod`：查找范围仅限==本类==，但可以找到==所有访问级别==（public, protected, default, private）的方法。
> * `getMethod`：查找范围包括==本类及父类/接口==，但==只能==找到 ` ==public==  方法。
> * 在需要调用特定类的（包括非 public）方法时，通常使用 `getDeclaredMethod`。

## 2. 核心用法：`invoke()` 动态调用

`Method` 对象最核心的方法是 `invoke(Object obj, Object... args)`，它允许你动态地执行该方法。

> [!EXAMPLE] 使用 invoke()
> ```java
> Employee employee = new Employee();
> try {
>     Method setNameMethod = Employee.class.getDeclaredMethod("setName", String.class);
>
>     // 动态调用 employee 对象的 setName 方法，传入参数 "张三"
>     // 等价于 employee.setName("张三");
>     setNameMethod.invoke(employee, "张三");
>
>     Method getNameMethod = Employee.class.getDeclaredMethod("getName");
>     // 调用无参方法时，第二个参数省略
>     Object nameValue = getNameMethod.invoke(employee); // 返回值是 Object 类型
>     System.out.println(nameValue); // 输出 "张三"
>
>     // 如果调用静态方法，第一个参数 obj 传入 null
>     // Method staticMethod = ...
>     // staticMethod.invoke(null, args...);
>
> } catch (NoSuchMethodException | IllegalAccessException | InvocationTargetException e) {
>     // 处理可能发生的异常
>     // IllegalAccessException: 如果方法是 private 且未设置 setAccessible(true)
>     // InvocationTargetException: 如果被调用的方法内部抛出了异常
>     e.printStackTrace();
> }
> ```

> [!NOTE] `invoke()` 方法解析
> * **`obj` (第一个参数)**：指定要在==哪个对象实例上==调用该方法。如果调用的是==静态方法==，则传入 `null`。
> * **`args` (可变参数)**：按顺序提供调用该方法所需的==实际参数值==。如果方法无参数，则省略此部分。参数的数量和类型必须与获取 `Method` 对象时指定的 `parameterTypes` 严格匹配。
> * **返回值**: `invoke()` 方法的返回值是 `Object` 类型，表示被调用方法的实际返回值。如果被调用方法返回 `void`，则 `invoke()` 返回 `null`。如果返回基本类型，会被自动装箱成对应的包装类。

## 3. 其他常用方法

* **`getName()`**: 获取方法名 (String)。
* **`getReturnType()`**: 获取方法的返回类型 (Class<?>).
   * ** getParameterTypes()**: 获取方法参数类型列表 (Class<?>[]).
* **`getModifiers()`**: 获取方法的修饰符 (int，需要使用 `Modifier` 类解析)。
* **`getAnnotation(Class<A> annotationClass)`**: 获取方法上的指定类型的注解。
* **`getDeclaredAnnotations()`**: 获取方法上声明的所有注解。
* **`setAccessible(boolean flag)`**: ==暴力破解==访问权限。如果 `flag` 为 `true`，则允许调用 `private` 方法或访问 `private` 字段（需要配合 `Field` 对象）。**谨慎使用！**

## 4. 在项目中的应用

`Method` 对象在 `AutoFillAspect` 中扮演了核心角色。

> [!NOTE] 应用场景分析
> 1.  **获取 Setter 方法**：在 [[项目实践/苍穹外卖/公共字段自动填充/具体实现/5.反射调用|5.反射调用]] 中，使用 `entity.getClass().getDeclaredMethod(methodName, paramType)` (其中 `methodName` 来自 `AutoFillConstant`) 来获取 `setCreateTime`, `setUpdateTime` 等方法的 `Method` 对象。
> 2.  **动态调用 Setter**：获取到 `Method` 对象后，使用 `method.invoke(entity, value)` 将当前时间 (`now`) 或用户ID (`currentUserId`) 动态地设置到 `entity` 对象中。
> 3.  **读取注解**：在 [[项目实践/苍穹外卖/公共字段自动填充/具体实现/4.实现AOP通知(before)|4.实现AOP通知(before)]] 中，先通过 `methodSignature.getMethod()` 获取被拦截方法的 `Method` 对象，然后调用 `method.getAnnotation(AutoFill.class)` 来读取 `@AutoFill` 注解。

---
**返回MOC：**
[[知识库/java基础/Java-反射/Java-反射 (MOC)|Java-反射 (MOC)]]