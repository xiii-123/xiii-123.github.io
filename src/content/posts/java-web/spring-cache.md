---
title: Spring Cache 使用指南
published: 2025-04-24
description: '使用Spring Cache 进行Java应用程序的缓存管理'
image: ''
tags: [Java, Spring, 缓存]
category: 'Java'
draft: false 
---
Spring Cache 是 Spring 框架提供的一种声明式缓存抽象机制，它通过注解的方式简化了缓存的实现和使用，适用于多种缓存场景。以下是对 Spring Cache 的详细介绍：
### 1. 什么是 Spring Cache？
Spring Cache 是一种缓存抽象机制，它通过注解（如 `@Cacheable`、`@CacheEvict` 和 `@CachePut`）和 AOP（面向切面编程）技术，为应用程序提供透明的缓存支持。这种机制的核心目标是将缓存逻辑与应用业务逻辑分离，从而简化开发过程。
- **核心思想**：通过方法级别的缓存注解，Spring Cache 可以自动缓存方法的输入和输出，避免重复执行耗时的计算或数据库查询。
- **优点**：
  - **简化开发**：业务代码无需关心底层的缓存实现细节。
  - **灵活性**：支持多种缓存框架（如 EhCache、Redis、Caffeine 等）。
  - **透明性**：缓存逻辑对开发者透明，不影响原有代码结构。
---
### 2. 核心注解及功能
Spring Cache 提供了以下核心注解，用于实现缓存功能：
1. **`@Cacheable`**  
   - **功能**：用于将方法的返回值缓存起来。当方法再次被调用时，如果缓存中存在对应的值，则直接返回缓存结果，而不执行方法体。
   - **使用场景**：适用于数据库查询或计算密集型任务。
2. **`@CacheEvict`**  
   - **功能**：用于从缓存中移除指定的数据。
   - **使用场景**：例如在数据更新或删除操作后，需要清除相关缓存。
3. **`@CachePut`**  
   - **功能**：用于将方法的返回值放入缓存中，而不影响方法的执行。
   - **使用场景**：适用于数据更新场景，确保缓存中的数据与数据库保持一致。
4. **`@Caching`**  
   - **功能**：组合多个缓存操作（如多个 `@Cacheable`、`@CacheEvict` 或 `@CachePut`）。
   - **使用场景**：需要同时执行多个缓存操作的场景。
---
### 3. 工作原理
Spring Cache 的工作原理主要基于 AOP 和缓存管理器（`CacheManager`）：
1. **AOP 支持**  
   - Spring Cache 通过 AOP 拦截带有缓存注解的方法，并在方法执行前后插入缓存逻辑。
   - 当方法被调用时，Spring Cache 会检查缓存中是否存在结果。如果存在，则直接返回缓存结果；如果不存在，则执行方法并将结果存入缓存。
2. **缓存管理器（`CacheManager`）**  
   - `CacheManager` 负责创建和管理缓存实例（`Cache`），并支持多种缓存实现（如 RedisCacheManager、EhCacheCacheManager 等）。
   - `Cache` 负责缓存的读写操作。
3. **缓存抽象**  
   - Spring Cache 提供了统一的缓存接口（`org.springframework.cache.Cache`），使得开发者可以无缝切换不同的缓存实现。
---

### 4. 示例代码
以下是一个基于 Spring Cache + Redis 的实际使用案例，包含配置和代码实现：

---

#### 1. 添加依赖 (pom.xml)
```xml
<!-- Spring Boot Cache -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>

<!-- Redis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

```

---

#### 2. 配置 Redis 连接 (application.yml)
```yaml
spring:
    redis:
        host: localhost
        port: 6379
        password: 123456
        database: 1
```

---

### 3. 开启缓存
```java
@Slf4j
@SpringBootApplication
@EnableCaching
public class CacheDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(CacheDemoApplication.class,args);
        log.info("项目启动成功...");
    }
}
```

---

### 4. Controller 层使用缓存注解
```java
@Service
public class Controller {

    // 模拟数据库访问
    @Autowired
    private UserRepository userRepository;

    // 缓存结果，key为方法名+参数id
    @Cacheable(cacheName = "userCache", value = "users", key = "#id")
    public User getUserById(Long id) {
        System.out.println("查询数据库用户：" + id);
        return userRepository.findById(id).orElse(null);
    }

    // 更新缓存
    @CachePut(cacheName = "userCache", value = "users", key = "#user.id")
    public User updateUser(User user) {
        userRepository.save(user);
        return user;
    }

    // 删除缓存
    @CacheEvict(cacheName = "userCache", value = "users", key = "#id")
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
}
```

---

### 5. 缓存注解说明
| 注解          | 作用                                 |
|---------------|--------------------------------------|
| `@Cacheable`  | 方法结果存入缓存，下次直接读取缓存   |
| `@CachePut`   | 更新缓存（通常用于修改操作）         |
| `@CacheEvict` | 删除缓存（通常用于删除操作）         |
| `@Caching`    | 组合多个缓存操作                     |
---
