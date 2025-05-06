---
title: mybatis-plus
published: 2025-05--06
description: 'mybatis-plus的基本使用'
image: ''
tags: [mybatis-plus]
category: 'java-cloud'
draft: false 
---

MyBatis-Plus 是 MyBatis 的一个增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。下面介绍 MyBatis-Plus 中常见的注解及其用法，以及常见的条件构造器及其用法。
### 常见注解及其用法
1. **@TableName**
   - 用于指定表名。
   - 示例：
     ```java
     @TableName("user")
     public class User {
     }
     ```
2. **@TableId**
   - 用于指定表的主键。
   - 示例：
     ```java
     @TableId(value = "id", type = IdType.AUTO)
     private Long id;
     ```
3. **@TableField**
   - 用于指定数据库字段与实体类字段的映射关系。
   - 示例：
     ```java
     @TableField("user_name")
     private String userName;
     ```
4. **@Version**
   - 用于乐观锁版本号。
   - 示例：
     ```java
     @Version
     private Integer version;
     ```
5. **@LogicDelete**
   - 用于逻辑删除。
   - 示例：
     ```java
     @TableLogic
     private Integer deleted;
     ```
6. **@Interceptor**
   - 用于定义拦截器。
   - 示例：
     ```java
     @Intercepts({@Signature(type = Executor.class, method = "update", args = {MappedStatement.class, Object.class})})
     public class MyInterceptor implements Interceptor {
     }
     ```
7. **@Select**
   - 用于定义查询语句。
   - 示例：
     ```java
     @Select("SELECT * FROM user WHERE id = #{id}")
     User getUserById(@Param("id") Long id);
     ```
8. **@Insert**
   - 用于定义插入语句。
   - 示例：
     ```java
     @Insert("INSERT INTO user(user_name, age) VALUES(#{userName}, #{age})")
     int insertUser(@Param("userName") String userName, @Param("age") Integer age);
     ```
9. **@Update**
   - 用于定义更新语句。
   - 示例：
     ```java
     @Update("UPDATE user SET user_name = #{userName} WHERE id = #{id}")
     int updateUser(@Param("id") Long id, @Param("userName") String userName);
     ```
10. **@Delete**
    - 用于定义删除语句。
    - 示例：
      ```java
      @Delete("DELETE FROM user WHERE id = #{id}")
      int deleteUser(@Param("id") Long id);
      ```
### 常见条件构造器及其用法
MyBatis-Plus 提供了多种条件构造器，最常用的是 `QueryWrapper` 和 `UpdateWrapper`。
1. **QueryWrapper**
   - 用于构建查询条件。
   - 示例：
     ```java
     QueryWrapper<User> queryWrapper = new QueryWrapper<>();
     queryWrapper.eq("user_name", "张三").between("age", 20, 30);
     List<User> users = userMapper.selectList(queryWrapper);
     ```
2. **UpdateWrapper**
   - 用于构建更新条件。
   - 示例：
     ```java
     UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();
     updateWrapper.eq("user_name", "张三").set("age", 31);
     int result = userMapper.update(null, updateWrapper);
     ```
3. **LambdaQueryWrapper**
   - 使用 Lambda 表达式构建查询条件，避免硬编码字段名。
   - 示例：
     ```java
     LambdaQueryWrapper<User> lambdaQuery = Wrappers.lambdaQuery();
     lambdaQuery.eq(User::getUserName, "张三").between(User::getAge, 20, 30);
     List<User> users = userMapper.selectList(lambdaQuery);
     ```
4. **LambdaUpdateWrapper**
   - 使用 Lambda 表达式构建更新条件。
   - 示例：
     ```java
     LambdaUpdateWrapper<User> lambdaUpdate = Wrappers.lambdaUpdate();
     lambdaUpdate.eq(User::getUserName, "张三").set(User::getAge, 31);
     int result = userMapper.update(null, lambdaUpdate);
     ```

### 自定义 SQL
MyBatis-Plus 允许开发者在使用它提供的强大功能的同时，也能够自定义 SQL 来满足一些特殊的查询需求。自定义 SQL 的使用场景通常包括：
1. **复杂查询**：当 MyBatis-Plus 提供的查询构造器无法满足复杂的查询需求时，可以使用自定义 SQL。
2. **性能优化**：对于一些需要优化执行计划的查询，自定义 SQL 可以让开发者更精细地控制查询过程。
3. **数据库特性**：使用特定数据库的特性，如 SQL Server 的 `ROW_NUMBER()` 或 MySQL 的 `GROUP_CONCAT()` 等。
4. **多表关联**：处理复杂的表关联查询，尤其是在涉及多个表和复杂关联条件时。
#### 常见用法
1. **在Service中构造条件**：
   此处使用Test演示
   ```java
   @Test
    void testCustomSqlUpdate() {
        List<Long> ids = List.of(1L,2L,3L);
        int amount = 200;

        QueryWrapper<User> wrapper = new QueryWrapper<User>().in("id", ids);

        userMapper.updateBalanceByIds(wrapper, amount);
    }
   ```
2. **构造新的Mapper接口方法**：
   ```java
   public interface UserMapper extends BaseMapper<User> {

    void updateBalanceByIds(@Param(Constants.WRAPPER) QueryWrapper<User> wrapper, @Param("amount") int amount);
   }
   ```
   注意此处的注解`@Param(Constants.WRAPPER)`，它表示将QueryWrapper对象作为参数传递给SQL语句。

3. **在Mapper.xml接口中使用传进来的Wrapper简化SQL代码**：
   ```xml
   <update id="updateBalanceByIds">
        update user set balance = balance - #{amount} ${ew.customSqlSegment}
    </update>
   ```


