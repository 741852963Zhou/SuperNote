
`ThreadLocal` 是 Java `java.lang` 包提供的一个类，它提供了一种 **==线程局部变量==** (Thread-Local Variables) 的机制。

## 1. 核心概念

> [!NOTE] 核心解析
> `ThreadLocal` 的核心思想是：为使用该变量的==每个线程==都提供==一个独立的变量副本==。
>
> * **==线程隔离==**：每个线程只能访问和修改自己的副本，无法访问其他线程的副本。这使得 `ThreadLocal` 成为在多线程环境下解决**==线程安全==**问题的一种有效手段，特别是对于那些需要在线程内部共享、但又希望避免线程间干扰的状态信息（如用户身份、事务ID等）。
> * **==数据传递==**：它还可以在同一个线程的**不同调用层级**之间传递数据，而==无需通过方法参数==显式传递。

## 2. 工作原理 (简化理解)

可以想象每个 `Thread` 对象内部都有一个类似 `Map` 的结构 (实际上是 `ThreadLocalMap`)，这个 Map 的 Key 是 `ThreadLocal` 对象本身，Value 就是这个线程对应的变量副本。

* 当调用 `threadLocal.set(value)` 时，实际上是获取==当前线程==，然后在当前线程的 "Map" 中放入 `{ threadLocal对象: value }` 这个键值对。
* 当调用 `threadLocal.get()` 时，是获取==当前线程==，然后从当前线程的 "Map" 中根据 `threadLocal` 对象这个 Key 来查找对应的 Value。
* 当调用 `threadLocal.remove()` 时，是获取==当前线程==，然后从当前线程的 "Map" 中移除 `threadLocal` 对象对应的键值对。

## 3. 基本用法

> [!EXAMPLE] 常用方法
> ```java
> // 1. 创建一个 ThreadLocal 变量 (通常是 static final)
> private static final ThreadLocal<String> userContext = new ThreadLocal<>();
>
> // 2. 在线程的某个点设置值
> userContext.set("当前登录用户 A");
>
> // 3. 在线程的另一个点获取值
> String currentUser = userContext.get(); // currentUser 的值是 "当前登录用户 A"
>
> // 4. 在线程结束前移除值 (非常重要！)
> userContext.remove();
> ```

## 4. 内存泄漏风险与 `remove()`

> [!WARNING] 必须调用 `remove()`
> 在使用==线程池==的环境下（例如 Web 服务器如 Tomcat），线程是会被**==复用==**的。
>
> 如果一个线程处理完请求后，**没有调用 `threadLocal.remove()`** 来清理其副本数据，那么当这个线程被**下一个请求**复用时，`threadLocal.get()` 可能会获取到**==上一个请求残留的数据==**！这会导致严重的数据污染和安全问题。
>
> 同时，如果 `ThreadLocal` 存储的对象不再被使用，但 `ThreadLocalMap` 中还持有对它的引用（并且 Key `ThreadLocal` 对象本身还存在强引用），就可能导致==内存泄漏==。
>
> **==最佳实践==**：始终在代码的 `finally` 块中，或者使用 `Filter`/`Interceptor` 的 `afterCompletion` 等机制，确保调用 `remove()` 方法来清理 `ThreadLocal` 变量。

## 5. 在项目中的应用

在“苍穹外卖”项目中，`ThreadLocal` 被用来在 `JwtTokenAdminInterceptor` 和 `AutoFillAspect` 之间传递当前登录用户的ID。

> [!NOTE] 应用场景分析
> 1.  **创建工具类**：`BaseContext` 类封装了一个 `public static ThreadLocal<Long> threadLocal` 实例，并提供了 `setCurrentId`, `getCurrentId`, `removeCurrentId` 三个静态方法。
> 2.  **拦截器 `set` 值**：在 `JwtTokenAdminInterceptor` 的 `preHandle` 方法中，成功解析 JWT 获取 `empId` 后，调用 `BaseContext.setCurrentId(empId)` 将 ID 存入当前线程。
> 3.  **切面 `get` 值**：在 `AutoFillAspect` 的 `before` 通知中，通过调用 `BaseContext.getCurrentId()` 获取之前存入的用户 ID，用于填充 `createUser` 和 `updateUser` 字段。
> 4.  **风险点**：项目中似乎缺少在请求结束后调用 `BaseContext.removeCurrentId()` 的逻辑，这可能导致线程复用时的数据污染或内存泄漏。

---
**相关链接：**
[[项目实践/苍穹外卖/公共字段自动填充/具体实现/6.ThreadLocal传递用户ID|6.ThreadLocal传递用户ID]]