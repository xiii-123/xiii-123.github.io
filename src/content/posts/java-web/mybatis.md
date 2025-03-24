---
title: mybatis入门
published: 2025-03-21
description: 'mybatis的基本使用'
image: ''
tags: [mybatis]
category: 'java-web'
draft: false 
---
MyBatis 是一个流行的 Java 持久层框架，它简化了数据库操作并提供了一种将 SQL 语句与 Java 代码分离的方法。MyBatis 支持注解方式来配置映射，这使得配置更加简洁。

## 常用注解
1. `@Select`：用于定义查询语句。
2. `@Insert`：用于定义插入语句。
3. `@Update`：用于定义更新语句。
4. `@Delete`：用于定义删除语句。
5. `@Param`：用于给方法参数命名，以便在 SQL 语句中使用。
6. `@Results`：用于定义映射结果集。
7. `@Result`：用于定义结果集的映射关系。
8. `@One`：用于一对一关联查询。
9. `@Many`：用于一对多关联查询。
## 示例代码：
```java
import org.apache.ibatis.annotations.*;
import java.util.List;
public interface UserMapper {
    // 查询所有用户
    @Select("SELECT * FROM users")
    List<User> getAllUsers();
    // 根据用户ID查询用户
    @Select("SELECT * FROM users WHERE id = #{id}")
    User getUserById(@Param("id") int id);
    // 插入新用户
    @Insert("INSERT INTO users(username, password) VALUES(#{username}, #{password})")
    @Options(useGeneratedKeys = true, keyProperty = "id")
    int insertUser(User user);
    // 更新用户信息
    @Update("UPDATE users SET username = #{username}, password = #{password} WHERE id = #{id}")
    int updateUser(User user);
    // 删除用户
    @Delete("DELETE FROM users WHERE id = #{id}")
    int deleteUser(@Param("id") int id);
    // 查询用户及其关联的订单
    @Select("SELECT * FROM users WHERE id = #{id}")
    @Results({
        @Result(property = "id", column = "id"),
        @Result(property = "orders", column = "id", javaType = List.class,
                many = @Many(select = "com.example.mapper.OrderMapper.getOrdersByUserId"))
    })
    User getUserWithOrders(@Param("id") int id);
}
public class User {
    private int id;
    private String username;
    private String password;
    // getter and setter methods
}
public class Order {
    private int id;
    private int userId;
    // other fields, getter and setter methods
}
public interface OrderMapper {
    // 根据用户ID查询订单
    @Select("SELECT * FROM orders WHERE user_id = #{userId}")
    List<Order> getOrdersByUserId(@Param("userId") int userId);
}
```
在上述示例中，`UserMapper` 接口定义了多种数据库操作，包括查询、插入、更新和删除。`@Select`、`@Insert`、`@Update` 和 `@Delete` 注解分别用于指定相应的 SQL 语句。`@Param` 注解用于给方法参数命名，以便在 SQL 语句中引用。`@Results` 和 `@Result` 注解用于定义查询结果的映射关系，而 `@One` 和 `@Many` 注解用于处理关联查询。
请注意，为了使这些注解工作，你需要在 MyBatis 配置文件中启用注解支持，或者使用 MyBatis-Spring 时在 Spring 配置中扫描映射器接口。

注：当使用springboot官方骨架时，不用显示使用`@Param`注解。

## 使用`mapper.xml`
使用 MyBatis 的 `mapper.xml` 文件编写 SQL 语句并与 Mapper 接口链接是一种常见的做法，它允许你将 SQL 语句与 Java 代码分离，提高可维护性。以下是如何使用 `mapper.xml` 文件编写 SQL 并与 Mapper 类链接的步骤：
### 1. 创建 Mapper 接口
首先，创建一个 Mapper 接口，该接口定义了用于数据库操作的方法。
```java
public interface UserMapper {
    User getUserById(int id);
    List<User> getAllUsers();
    // 其他数据库操作方法
}
```
### 2. 创建 mapper.xml 文件
在相同的包或资源目录下创建一个 `mapper.xml` 文件，该文件包含 SQL 语句和映射配置。
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mapper.UserMapper">
    <!-- 定义结果映射 -->
    <resultMap id="userResultMap" type="com.example.model.User">
        <id property="id" column="id" />
        <result property="username" column="username" />
        <result property="password" column="password" />
        <!-- 其他字段映射 -->
    </resultMap>
    <!-- 查询用户通过ID -->
    <select id="getUserById" resultMap="userResultMap">
        SELECT id, username, password FROM users WHERE id = #{id}
    </select>
    <!-- 查询所有用户 -->
    <select id="getAllUsers" resultMap="userResultMap">
        SELECT id, username, password FROM users
    </select>
    <!-- 其他SQL语句 -->
</mapper>
```
### 3. 配置 MyBatis
如果`mapper.xml`文件在resources目录下，则无需配置，否则需要在 MyBatis 配置文件中指定 `mapper.xml` 文件的路径。
```yml
mybatis:
  mapper-locations: classpath:mapper/*.xml
```
### 4. 使用 Mapper 接口
最后，在你的 Java 代码中使用 Mapper 接口进行数据库操作。
```java
public class UserService {
    private UserMapper userMapper;
    // 通过构造函数或setter方法注入UserMapper

    public User getUserById(int id) {
        return userMapper.getUserById(id);
    }

    public List<User> getAllUsers() {
        return userMapper.getAllUsers();
    }
    // 其他数据库操作方法
}
```

## 动态SQL
MyBatis 的动态 SQL 是其强大特性之一，它允许你在 XML 映射文件或注解中编写包含逻辑条件的 SQL 语句，从而根据不同的条件动态生成 SQL。这种特性使得 SQL 语句更加灵活，能够适应多种查询需求。以下是 MyBatis 中常用的动态 SQL 元素：
### 1. `<if>`
`<if>` 标签用于判断条件是否成立，如果成立，则执行标签内的 SQL 片段。
```xml
<select id="selectByCondition" resultType="User">
  SELECT * FROM users
  <where>
    <if test="name != null">
      AND name = #{name}
    </if>
    <if test="age != null">
      AND age = #{age}
    </if>
  </where>
</select>
```
### 2. `<choose>`, `<when>`, `<otherwise>`
这三个标签组合起来，类似于 Java 中的 `switch` 语句，用于选择多个条件中的一个执行。
```xml
<select id="selectByGrade" resultType="User">
  SELECT * FROM users
  <where>
    <choose>
      <when test="grade == 'A'">
        AND score > 90
      </when>
      <when test="grade == 'B'">
        AND score BETWEEN 80 AND 90
      </when>
      <otherwise>
        AND score < 80
      </otherwise>
    </choose>
  </where>
</select>
```
### 3. `<where>`
`<where>` 标签会自动处理 SQL 语句中的 `WHERE` 关键字，如果内部条件成立，则插入 `WHERE`，否则不插入。
```xml
<!-- 示例同上 -->
```
### 4. `<set>`
`<set>` 标签用于更新操作，它会自动处理 SQL 语句中的 `SET` 关键字，并会智能地去除多余的逗号。
```xml
<update id="updateUser">
  UPDATE users
  <set>
    <if test="name != null">
      name = #{name},
    </if>
    <if test="age != null">
      age = #{age},
    </if>
  </set>
  WHERE id = #{id}
</update>
```
### 5. `<foreach>`
`<foreach>` 标签用于遍历集合，通常用于 `IN` 查询或者批量插入。
```xml
<select id="selectByIds" resultType="User">
  SELECT * FROM users
  WHERE id IN
  <foreach item="id" collection="list" open="(" separator="," close=")">
    #{id}
  </foreach>
</select>
```
### 6. `<bind>`
`<bind>` 标签用于创建一个变量，并将其绑定到上下文中。
```xml
<select id="selectByPattern" resultType="User">
  <bind name="pattern" value="'%' + _parameter.pattern + '%'"/>
  SELECT * FROM users
  WHERE name LIKE #{pattern}
</select>
```
### 使用动态 SQL 的注意事项：
- 动态 SQL 会增加 SQL 语句的复杂度，因此需要仔细测试以确保生成的 SQL 语句正确。
- 避免在动态 SQL 中使用复杂的逻辑，以免影响可读性和维护性。
- 使用动态 SQL 时，要注意防止 SQL 注入攻击，尽量使用参数化查询。
通过合理使用这些动态 SQL 元素，可以大大提高 MyBatis 的灵活性和适用性，使得 SQL 语句能够根据不同的条件动态生成，满足复杂的查询需求。

## 主键返回
在 MyBatis 中，当你插入一条新记录时，通常需要获取数据库自动生成的主键值。MyBatis 提供了多种方式来返回主键值，以下是几种常见的方法：
### 1. 使用 `useGeneratedKeys` 和 `keyProperty`
在 MyBatis 的映射文件中，你可以在 `insert` 标签中使用 `useGeneratedKeys` 属性来告诉 MyBatis 你想要获取自动生成的主键值，并通过 `keyProperty` 属性指定哪个字段是主键。
```xml
<insert id="insertUser" useGeneratedKeys="true" keyProperty="id">
  INSERT INTO users (name, age) VALUES (#{name}, #{age})
</insert>
```
在上面的例子中，`users` 表的 `id` 字段是自动生成的主键。当你执行这个 `insert` 操作时，MyBatis 会将生成的 `id` 值设置到传入的 `User` 对象的 `id` 属性中。
### 2. 使用 `selectKey`
如果你使用的数据库不支持自动生成主键的返回，或者你需要更复杂的逻辑来获取主键，可以使用 `selectKey` 标签。
```xml
<insert id="insertUser">
  <selectKey keyProperty="id" resultType="int" order="AFTER">
    SELECT LAST_INSERT_ID()
  </selectKey>
  INSERT INTO users (name, age) VALUES (#{name}, #{age})
</insert>
```
在这个例子中，`selectKey` 标签中的 SQL 语句会在插入操作之后执行，以获取最后插入的记录的主键值。`keyProperty` 指定将主键值设置到对象的哪个属性，`resultType` 指定主键的类型，`order` 属性指定 `selectKey` 的执行时机（`BEFORE` 或 `AFTER`）。
### 3. 注解方式
如果你使用的是基于注解的 MyBatis 配置，也可以通过注解来返回主键。
```java
@Insert("INSERT INTO users (name, age) VALUES (#{name}, #{age})")
@Options(useGeneratedKeys = true, keyProperty = "id")
int insertUser(User user);
```
在这个例子中，`@Options` 注解的 `useGeneratedKeys` 和 `keyProperty` 属性与 XML 配置中的含义相同。
### 注意事项：
- `useGeneratedKeys` 和 `keyProperty` 属性通常用于支持自动生成主键的数据库，如 MySQL。
- `selectKey` 更为通用，可以用于任何数据库，但需要手动编写获取主键的 SQL 语句。
- 确保你的 MyBatis 版本支持你使用的特性，因为不同的版本可能有不同的行为和限制。
通过这些方法，MyBatis 可以方便地返回插入操作后数据库自动生成的主键值，从而使得应用程序能够使用这些主键值进行后续的操作。

## ResultMap
在MyBatis中，`resultMap` 是一个非常强大的工具，用于解决数据库查询结果与Java对象之间的映射问题，特别是对于连表查询等复杂场景。下面我将通过一个示例来展示如何使用 `resultMap` 来查询员工及其工作经历，其中员工类中包含一个工作经历列表。
### 示例场景
假设我们有两个表：`employee`（员工表）和 `work_experience`（工作经历表）。`employee` 表存储员工的基本信息，`work_experience` 表存储员工的工作经历，并且通过 `employee_id` 与 `employee` 表关联。
**员工表（employee）：**
- id
- name
- age
**工作经历表（work_experience）：**
- id
- employee_id
- company
- position
- start_date
- end_date
**员工类（Employee）：**
```java
public class Employee {
    private Integer id;
    private String name;
    private Integer age;
    private List<WorkExperience> workExperiences;
    // getters and setters
}
```
**工作经历类（WorkExperience）：**
```java
public class WorkExperience {
    private Integer id;
    private Integer employeeId;
    private String company;
    private String position;
    private Date startDate;
    private Date endDate;
    // getters and setters
}
```
### MyBatis配置
#### 1. 定义 resultMap
在MyBatis的映射文件中，我们首先定义两个 `resultMap`，一个用于 `Employee`，另一个用于 `WorkExperience`。
**Employee resultMap：**
```xml
<resultMap id="EmployeeResultMap" type="Employee">
    <id property="id" column="id"/>
    <result property="name" column="name"/>
    <result property="age" column="age"/>
    <collection property="workExperiences" ofType="WorkExperience" 
                <id property="id" column="id"/>
                <result property="employeeId" column="employee_id"/>
                <result property="company" column="company"/>
                <result property="position" column="position"/>
                <result property="startDate" column="start_date"/>
                <result property="endDate" column="end_date"/>
    </collection>
</resultMap>
```

#### 2. 定义查询语句
**查询员工信息：**
```xml
<select id="selectEmployeeById" resultMap="EmployeeResultMap">
    SELECT * FROM employee WHERE id = #{id}
</select>
```
### 使用方式
在Java代码中，通过MyBatis的SqlSession调用 `selectEmployeeById` 方法即可获取员工及其工作经历的信息：
```java
try (SqlSession session = sqlSessionFactory.openSession()) {
    EmployeeMapper mapper = session.getMapper(EmployeeMapper.class);
    Employee employee = mapper.selectEmployeeById(1);
    System.out.println(employee.getName());
    for (WorkExperience we : employee.getWorkExperiences()) {
        System.out.println(we.getCompany() + " - " + we.getPosition());
    }
}
```
这里假设 `EmployeeMapper` 是一个接口，其中定义了 `selectEmployeeById` 方法，并且通过MyBatis的自动映射机制与XML文件中的SQL语句关联。

