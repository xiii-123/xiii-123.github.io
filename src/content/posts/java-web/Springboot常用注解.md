---
title: Springboot常见注解
published: 2025-03-22
description: 'Springboot常见注解'
image: ''
tags: [springboot]
category: 'java-web'
draft: false 
---
## 形参注解
在Spring Boot的Controller中，常用的形参注解有以下几种：
1. `@RequestParam`：用于绑定请求参数到方法参数，可以设置参数的默认值以及是否必须。
   ```java
   public String hello(@RequestParam(value = "name", defaultValue = "World") String name) {
       return "Hello, " + name;
   }
   ```
2. `@PathVariable`：用于绑定URL中的路径变量到方法参数。
   ```java
   @GetMapping("/users/{id}")
   public User getUserById(@PathVariable Long id) {
       // ...
   }
   ```
3. `@RequestBody`：用于绑定请求体中的内容到方法参数，通常用于处理POST请求中的JSON数据。
   ```java
   @PostMapping("/users")
   public User createUser(@RequestBody User user) {
       // ...
   }
   ```
4. `@RequestHeader`：用于绑定请求头中的值到方法参数。
   ```java
   public String hello(@RequestHeader("User-Agent") String userAgent) {
       return "User Agent: " + userAgent;
   }
   ```
5. `@CookieValue`：用于绑定Cookie中的值到方法参数。
   ```java
   public String hello(@CookieValue("JSESSIONID") String sessionId) {
       return "Session ID: " + sessionId;
   }
   ```
6. `@ModelAttribute`：用于绑定请求参数到Java对象，通常用于表单提交。
   ```java
   @PostMapping("/users")
   public User createUser(@ModelAttribute User user) {
       // ...
   }
   ```
7. `@SessionAttribute`：用于绑定 HttpSession 中的属性到方法参数。
   ```java
   public String hello(@SessionAttribute("user") User user) {
       return "Hello, " + user.getName();
   }
   ```
8. `@MatrixVariable`：用于绑定URL中的矩阵变量到方法参数。
   ```java
   @GetMapping("/cars/{make}/{model}")
   public Car getCar(@PathVariable String make, @PathVariable String model, @MatrixVariable(name="color") String color) {
       // ...
   }
   ```
9. `@Value`：虽然不常用于Controller方法参数，但可以用于注入配置文件中的值。
   ```java
   public String hello(@Value("${app.name}") String appName) {
       return "App Name: " + appName;
   }
   ```

## RESTful mapping注解
在Spring Boot中，用于创建RESTful Web服务的常见注解被称为RESTful mapping注解。这些注解主要用于控制器（Controller）类的方法上，以定义HTTP请求的映射。以下是一些常见的RESTful mapper注解：
1. @GetMapping：映射HTTP GET请求 onto specific handler methods.
	* 示例：`@GetMapping("/users")` 映射到GET /users的请求。
2. @PostMapping：映射HTTP POST请求 onto specific handler methods.
	* 示例：`@PostMapping("/users")` 映射到POST /users的请求。
3. @PutMapping：映射HTTP PUT请求 onto specific handler methods.
	* 示例：`@PutMapping("/users/{id}")` 映射到PUT /users/{id}的请求。
4. @DeleteMapping：映射HTTP DELETE请求 onto specific handler methods.
	* 示例：`@DeleteMapping("/users/{id}")` 映射到DELETE /users/{id}的请求。
5. @PatchMapping：映射HTTP PATCH请求 onto specific handler methods.
	* 示例：`@PatchMapping("/users/{id}")` 映射到PATCH /users/{id}的请求。
6. @RequestMapping：是一个通用的映射注解，可以用于映射所有类型的HTTP请求。你可以指定请求方法、路径、参数等。
	* 示例：`@RequestMapping(value = "/users", method = RequestMethod.GET)` 映射到GET /users的请求。

## Mybatis注解
在Spring Boot中，当涉及到数据库操作，尤其是使用MyBatis或MyBatis-Plus等ORM框架时，RESTful风格的mapper层注解通常指的是在Mapper接口中用于定义SQL操作的注解。这些注解不是Spring Boot特有的，而是由ORM框架提供的。以下是一些常见的RESTful风格的mapper层注解：
1. @Insert：用于定义插入操作。
   ```java
   @Insert("INSERT INTO users (name, age) VALUES (#{name}, #{age})")
   int insertUser(User user);
   ```
2. @Select：用于定义查询操作。
   ```java
   @Select("SELECT * FROM users WHERE id = #{id}")
   User selectUserById(@Param("id") int id);
   ```
3. @Update：用于定义更新操作。
   ```java
   @Update("UPDATE users SET name = #{name}, age = #{age} WHERE id = #{id}")
   int updateUser(User user);
   ```
4. @Delete：用于定义删除操作。
   ```java
   @Delete("DELETE FROM users WHERE id = #{id}")
   int deleteUserById(@Param("id") int id);
   ```
5. @Options：用于获取插入数据后生成的主键值。
   ```java
   @Insert("INSERT INTO users (name, age) VALUES (#{name}, #{age})")
   @Options(useGeneratedKeys = true, keyProperty = "id")
   int insertUser(User user);
   ```
6. @SelectKey：用于在插入操作后返回特定的键值。
   ```java
   @Insert("INSERT INTO users (name, age) VALUES (#{name}, #{age})")
   @SelectKey(statement = "CALL IDENTITY()", keyProperty = "id", before = false, resultType = Integer.class)
   int insertUser(User user);
   ```
7. @Param：用于给方法参数命名，以便在SQL语句中引用。
   ```java
   User selectUserByNameAndAge(@Param("name") String name, @Param("age") int age);
   ```
这些注解通常用于MyBatis的Mapper接口中，以简化SQL语句的编写和映射。需要注意的是，如果你使用的是JPA（Java Persistence API）或Spring Data JPA，那么你会使用不同的注解和方式来定义数据库操作，例如使用`@Query`注解来定义查询操作。
在Spring Data JPA中，你还可以使用预定义的方法命名约定来简化查询操作，而不需要显式使用注解。例如，通过定义一个方法名为`findByNameAndAge`，Spring Data JPA会自动生成相应的查询。

## 事务注解

Spring 框架提供了强大的事务管理功能，其中事务注解是简化事务管理的一种常用方式。Spring 的事务注解允许开发者以声明式的方式管理事务，无需编写复杂的事务管理代码。以下是 Spring 中常用的事务注解介绍：
### 1. @Transactional
`@Transactional` 是 Spring 中最常用的事务注解，用于声明事务边界。它可以应用于类或方法上。
- **作用于类**：表示该类中的所有公共方法都使用相同的事务配置。
- **作用于方法**：表示该方法使用特定的事务配置。
#### 主要属性：
- `value` 或 `transactionManager`：指定事务管理器。
- `propagation`：事务的传播行为，如 `REQUIRED`、`REQUIRES_NEW` 等。
- `isolation`：事务的隔离级别，如 `READ_COMMITTED`、`SERIALIZABLE` 等。
- `timeout`：事务的超时时间。
- `readOnly`：是否为只读事务。
- `rollbackFor`：指定哪些异常会导致事务回滚。
- `noRollbackFor`：指定哪些异常不会导致事务回滚。
### 示例：
```java
import org.springframework.transaction.annotation.Transactional;
@Transactional(propagation = Propagation.REQUIRED, isolation = Isolation.READ_COMMITTED)
public class UserService {
    @Transactional(readOnly = true)
    public User getUserById(Long id) {
        // ...
    }
    @Transactional(rollbackFor = Exception.class)
    public void updateUser(User user) {
        // ...
    }
}
```
### 2. @EnableTransactionManagement
`@EnableTransactionManagement` 是一个配置类级别的注解，用于启用 Spring 的注解驱动事务管理功能。通常用于主配置类或启动类上。
### 示例：
```java
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.annotation.EnableTransactionManagement;
@Configuration
@EnableTransactionManagement
public class AppConfig {
    // ...
}
```
### 注意事项：
- **事务管理器**：确保已经配置了相应的事务管理器，如 `DataSourceTransactionManager`、`HibernateTransactionManager` 等。
- **代理模式**：Spring 事务注解默认使用 JDK 动态代理，如果需要使用 CGLIB 代理，可以通过配置 `proxyTargetClass` 属性为 `true`。
- **方法可见性**：只有公共方法才能被 `@Transactional` 注解标记，否则事务配置不会生效。
- **异常处理**：事务的回滚依赖于异常的抛出，确保方法中抛出了正确的异常。
通过使用这些注解，开发者可以轻松地在 Spring 应用中管理事务，提高开发效率和代码的可维护性。


## 全局异常处理
在Spring Boot中，全局异常处理器可以帮助你集中处理整个应用中抛出的异常，使得异常处理更加统一和优雅。`@RestControllerAdvice` 和 `@ExceptionHandler` 是实现这一功能的关键注解。
### @RestControllerAdvice
`@RestControllerAdvice` 是 `@ControllerAdvice` 的子注解，专门用于处理RESTful风格的控制器抛出的异常。它本身包含了 `@ResponseBody` 注解，因此处理异常后返回的结果会自动转换为JSON格式。
### @ExceptionHandler
`@ExceptionHandler` 用于标注处理特定异常的方法。你可以指定这个方法处理一种异常或多种异常。
### 使用步骤
1. **创建异常处理类**
   创建一个类，并使用 `@RestControllerAdvice` 注解标记这个类。
2. **定义异常处理方法**
   在这个类中，定义方法来处理特定的异常。使用 `@ExceptionHandler` 注解来指定这个方法处理的异常类型。
### 示例代码
以下是一个简单的示例，展示如何使用 `@RestControllerAdvice` 和 `@ExceptionHandler` 来处理全局异常：
```java
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler
    public Result handle(Exception e) {
        log.error("服务器异常", e);
        return Result.error("服务器异常");
    }

    @ExceptionHandler
    public Result handleDuplicateKeyException(DuplicateKeyException e) {
        log.error("服务器异常", e);
        String errMessage = e.getMessage();
        return Result.error(errMessage);
    }
}
```

