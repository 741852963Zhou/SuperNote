### 核心流程概览

一次完整的前后端数据交互，在后端主要由两个“翻译官”协作完成：

1. **Jackson**: 负责 **`JSON 字符串`** (前端) 与 **`Java 对象 (DTO)`** (Controller) 之间的相互转换。
    
2. **MyBatis**: 负责 **`Java 对象 (Entity)`** (Mapper) 与 **`SQL 数据`** (数据库) 之间的相互转换。
    
---

### Step 1: 从前端到 Controller (Jackson 的工作)

当前端发送一个 POST 请求，并携带 JSON 数据时，名字的对应关系发生在 Controller 层。

- **前端发送的 JSON Body**:

    ```json
    {
      "userName": "lucy",
      "userAge": 28
    }
    ```

- **后端接收的 Controller 方法**:

    ```java
    @PostMapping("/users")
    public Result createUser(@RequestBody UserDTO userDTO) {
        // ...
    }
    ```
    
- **用于接收的 DTO 类**:

    ```java
    public class UserDTO {
        private String userName;
        private Integer userAge;
        // Getters and Setters...
    }
    ```
    
- **名字如何对应**:
    
    - `@RequestBody` 注解指示 Spring MVC 使用 Jackson 来解析请求体中的 JSON。
        
    - Jackson 会读取 JSON 的 `key` (例如 `"userName"`)。
        
    - 然后，它会在 `UserDTO` 类中寻找一个**完全同名**的字段 (`private String userName;`)。
        
    - 最后，通过调用该字段的 `setter` 方法 (`setUserName(...)`) 将 JSON 中的 `value` 赋值给这个字段。
        
    - **规则**: 默认情况下，JSON 的 `key` 和 Java 类的字段名必须**精确匹配，包括大小写**。
        

---

### Step 2: 从 Service 到 Mapper/数据库 (MyBatis 的工作)

当业务逻辑处理完毕，Service 层调用 Mapper 接口与数据库交互时，MyBatis 开始工作。这正是驼峰命名配置发挥作用的地方。

#### 场景 A: 写操作 (Java 对象 -> SQL)

这个方向的映射比较直接。

- **Service 层调用**:
    
    ```java
    User userEntity = new User();
    userEntity.setUserName("lucy"); // Java 对象使用驼峰命名
    userEntity.setUserAge(28);
    userMapper.insert(userEntity);
    ```
    
- **Mapper XML**
    
    ```xml
    <insert id="insert">
        INSERT INTO users (user_name, user_age)
        VALUES (#{userName}, #{userAge})
    </insert>
    ```
    
- **名字如何对应**:
    
    - MyBatis 看到 XML 中的占位符 `#{userName}`。
        
    - 它会去传入的 `userEntity` 对象中寻找一个名为 `userName` 的属性，并调用其 `getter` 方法 (`getUserName()`) 来获取值。
        
    - **规则**: 在写操作中，XML 里的 **`#{...}` 占位符名称** 必须和 **Java 实体类的字段名** 保持一致。
        

#### 场景 B: 读操作 (SQL 结果 -> Java 对象) —— 驼峰命名配置的核心作用

这是理解驼峰命名配置的关键。

- **数据库表结构**:
    
    ```sql
    -- 列名是下划线命名 (snake_case)
    CREATE TABLE users (
        id INT PRIMARY KEY,
        user_name VARCHAR(50),
        user_age INT
    );
    ```
    
- **Java 实体类**:
    
    ```java
    public class User {
        private Long id;
        private String userName; // 字段是驼峰命名 (camelCase)
        private Integer userAge;
        // Getters and Setters...
    }
    ```
    
- **Mapper XML**:
    
    ```xml
    <select id="findById" resultType="com.example.User">
        SELECT id, user_name, user_age FROM users WHERE id = #{id}
    </select>
    ```
    
- **问题**: `SELECT` 语句返回的列名是 `user_name`，而 Java 实体类中对应的字段是 `userName`。**两者名字不匹配！**
    
- **驼峰命名配置的影响**: 这个配置就是为了解决上述问题。
    

    ```yaml
    # 在 application.yml 中
    mybatis:
      configuration:
        map-underscore-to-camel-case: true
    ```
    
    - **当配置为 `true` (开启)**:
        
        1. MyBatis 从数据库获取到列名 `user_name`。
            
        2. 它**自动将下划线命名翻译为驼峰命名**，即 `user_name` -> `userName`。
            
        3. 然后，它拿着翻译后的名字 `userName` 去 `User` 实体类中寻找对应的 `setter` 方法 (`setUserName(...)`) 并进行赋值。
            
        4. **结果**: 映射成功，`User` 对象的所有字段都被正确填充。
            
    - **当配置为 `false` (关闭或未配置)**:
        
        1. MyBatis 从数据库获取到列名 `user_name`。
            
        2. 它**不会进行任何翻译**。
            
        3. 它直接拿着 `user_name` 去 `User` 实体类中寻找 `setUser_name(...)` 方法。
            
        4. **结果**: 找不到该方法，映射失败。最终返回的 `User` 对象中，`userName` 字段的值为 `null`。这时，你就必须手动编写 `<resultMap>` 来显式声明映射关系。
            

### 总结

|阶段|负责框架|源数据|目标数据|映射规则 / 关键配置|
|---|---|---|---|---|
|**请求解析**|**Jackson**|前端 JSON `key`|Controller DTO 字段|默认：精确、大小写敏感匹配。|
|**数据持久化(写)**|**MyBatis**|Entity 字段名|Mapper XML `#{...}`|两者名称必须一致。|
|**数据查询(读)**|**MyBatis**|数据库列名 (下划线)|Entity 字段名 (驼峰)|**`map-underscore-to-camel-case: true`** 自动翻译。|