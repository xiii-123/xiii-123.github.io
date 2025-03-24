---
title: pageHelper使用
published: 2025-03-23
description: 'pageHelper使用'
image: ''
tags: [pageHelper]
category: 'java-web'
draft: false 
---

PageHelper 是一款 MyBatis 的分页插件，它可以简化 MyBatis 的分页操作，提高开发效率。以下是使用 PageHelper 的基本步骤：
### 1. 添加依赖
首先，你需要在项目的 `pom.xml` 文件中添加 PageHelper 的依赖：
```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>最新版本</version>
</dependency>
```
请替换 `最新版本` 为当前最新的 PageHelper 版本。
### 2. 配置插件
在 MyBatis 的配置文件（如 `mybatis-config.xml`）中配置 PageHelper 插件：
```xml
<configuration>
    <!-- 其他配置 -->
    <!-- 配置分页插件 -->
    <plugins>
        <plugin interceptor="com.github.pagehelper.PageInterceptor">
            <!-- 配置参数 -->
            <property name="helperDialect" value="mysql"/>
            <!-- 其他参数 -->
        </plugin>
    </plugins>
</configuration>
```
`helperDialect` 属性用于指定数据库类型，例如 `mysql`、`oracle` 等。
### 3. 使用分页
在 Service 层或 DAO 层的方法中，使用 PageHelper 开始分页：
```java
import com.github.pagehelper.PageHelper;
import com.github.pagehelper.PageInfo;
import java.util.List;
public class MyService {
    public PageInfo<MyEntity> selectByPage(int pageNum, int pageSize) {
        // 开启分页
        PageHelper.startPage(pageNum, pageSize);
        // 执行查询
        List<MyEntity> list = myMapper.selectEntities();
        // 使用 PageInfo 包装返回结果
        return new PageInfo<>(list);
    }
}
```
- `pageNum`：当前页码
- `pageSize`：每页显示的记录数
### 4. 映射器方法
确保你的 MyBatis 映射器接口中的方法符合分页查询的要求，例如：
```java
import org.apache.ibatis.annotations.Select;
import java.util.List;
public interface MyMapper {
    @Select("SELECT * FROM my_table")
    List<MyEntity> selectEntities();
}
```
### 5. 处理分页结果
在 Controller 层或其它需要处理分页结果的层，你可以直接使用 `PageInfo` 对象，它包含了分页信息，如总记录数、总页数等：
```java
public class MyController {
    @Autowired
    private MyService myService;
    public ResponseEntity<PageInfo<MyEntity>> getEntitiesByPage(int pageNum, int pageSize) {
        PageInfo<MyEntity> pageInfo = myService.selectByPage(pageNum, pageSize);
        return ResponseEntity.ok(pageInfo);
    }
}
```
### 注意事项
- `PageHelper.startPage()` 方法只能紧跟着查询方法调用，否则分页信息可能会丢失。
- `PageInfo` 对象包含了丰富的分页信息，可以根据需要选择性地使用。
- PageHelper 支持多种配置参数，可以根据具体需求进行配置。
通过以上步骤，你就可以在 MyBatis 项目中使用 PageHelper 进行分页查询了。PageHelper 还提供了更多高级功能，如动态分页、自定义分页插件等，可以根据项目需求进行探索和使用。
