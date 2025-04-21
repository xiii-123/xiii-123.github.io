---
title: SpringBoot 自动装配
published: 2025-03-27
description: 'SpringBoot 自动装配'
image: ''
tags: [自动装配, Spring]
category: 'java-web'
draft: false 
---

Spring Boot 的自动装配（Auto-Configuration）是其核心特性之一，旨在简化应用的配置过程。

## 实现自动装配的方案

让我们先看看有那些方案能够实现自动装配。

### `@Component`+`@ComponentScan`
我们可以在自己的模块中定义一些组件，然后通过 `@ComponentScan` 扫描这些组件，将其注册到 Spring 容器中。

然后在使用到这些组件时，在启动程序中加上`@CompinentScan`注解，使得Spring能够扫描到这些组件。
```java
@ComponentScan(basePackages = {"org.example", "com.example.utils"})
@SpringBootApplication
public class SpringbootMybatisQuickstartApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootMybatisQuickstartApplication.class, args);
    }

}
```

### `Import`导入
- 导入形式主要有以下几种：
  1. 导入普通类
  2. 导入配置类
  3. 导入ImportSelector接口实现类
  4. 使用第三方依赖提供的 @EnableXxxxx注解

#### 1. 导入普通类
```java
@Import({User.class, UserMapper.class})
@SpringBootApplication
public class SpringbootMybatisQuickstartApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootMybatisQuickstartApplication.class, args);
    }

}
```

#### 2. 导入配置类
```java
@Configuration
public class HeaderConfig {
    @Bean
    public HeaderParser headerParser(){
        return new HeaderParser();
    }

    @Bean
    public HeaderGenerator headerGenerator(){
        return new HeaderGenerator();
    }
}
```
```java
@Import(HeaderConfig.class)
@SpringBootApplication
public class SpringbootMybatisQuickstartApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootMybatisQuickstartApplication.class, args);
    }

}
```

#### 3. 导入ImportSelector接口实现类
```java
public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[]{"org.example.User", "org.example.UserMapper"};
    }
}
```
```java
@Import(MyImportSelector.class) // 导入ImportSelector接口实现类
@SpringBootApplication
public class SpringbootMybatisQuickstartApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootMybatisQuickstartApplication.class, args);
    }
}
```

#### 4. 使用第三方依赖提供的 @EnableXxxxx注解
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Import(MyImportSelector.class)//指定要导入哪些bean对象或配置类
public @interface EnableHeaderConfig { 
}
```
```java
@EnableHeaderConfig // 使用第三方依赖提供的 @EnableXxxxx注解
@SpringBootApplication
public class SpringbootMybatisQuickstartApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootMybatisQuickstartApplication.class, args);
    }

}
```

## Spring自动装配的实现原理

### 1. `@SpringBootApplication`注解
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
```
`@SpringBootApplication`注解是一个复合注解，包含了
`@SpringBootConfiguration`、`@EnableAutoConfiguration`和`@ComponentScan`三个注解。其中`@EnableAutoConfiguration`注解是自动装配的核心。

### 2. `@EnableAutoConfiguration`注解
`@EnableAutoConfiguration`注解是一个开启自动装配的注解，它会自动扫描项目中的依赖，根据依赖自动配置项目。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
```
`@EnableAutoConfiguration`注解中包含了`@Import(AutoConfigurationImportSelector.class)`注解。

#### `@Import(AutoConfigurationImportSelector.class)`注解
`@Import(AutoConfigurationImportSelector.class)`注解的作用是导入`AutoConfigurationImportSelector`类，这个类会根据项目中的依赖自动配置项目。

```java
public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware,
		ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {
```
该类继承了`DeferredImportSelector`接口，该接口中有一个`selectImports`方法，该方法会返回一个数组，数组中包含了需要导入的类。
```java
@Override
public String[] selectImports(AnnotationMetadata annotationMetadata) {
    if (!isEnabled(annotationMetadata)) {
        return NO_IMPORTS;
    }
    AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
    return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
}
```
我们再来看`getAutoConfigurationEntry`。
```java
protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return EMPTY_ENTRY;
		}
		AnnotationAttributes attributes = getAttributes(annotationMetadata);
		List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
		configurations = removeDuplicates(configurations);
		Set<String> exclusions = getExclusions(annotationMetadata, attributes);
		checkExcludedClasses(configurations, exclusions);
		configurations.removeAll(exclusions);
		configurations = getConfigurationClassFilter().filter(configurations);
		fireAutoConfigurationImportEvents(configurations, exclusions);
		return new AutoConfigurationEntry(configurations, exclusions);
	}
```
`getCandidateConfigurations`方法会返回一个数组，数组中包含了需要导入的类。
```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
		ImportCandidates importCandidates = ImportCandidates.load(this.autoConfigurationAnnotation,
				getBeanClassLoader());
		List<String> configurations = importCandidates.getCandidates();
		Assert.notEmpty(configurations,
				"No auto configuration classes found in " + "META-INF/spring/"
						+ this.autoConfigurationAnnotation.getName() + ".imports. If you "
						+ "are using a custom packaging, make sure that file is correct.");
		return configurations;
	}
```
`ImportCandidates.load`方法会加载`META-INF/org.springframework.boot.autoconfigure.AutoConfiguration.imports`文件中的配置，该文件中包含了需要导入的类。

### 3. `META-INF/org.springframework.boot.autoconfigure.AutoConfiguration.imports`文件
`META-INF/org.springframework.boot.autoconfigure.AutoConfiguration.imports`文件中包含了需要导入的类，这些类会被自动装配到项目中。

```properties
org.mybatis.spring.boot.autoconfigure.MybatisLanguageDriverAutoConfiguration
org.mybatis.spring.boot.autoconfigure.MybatisAutoConfiguration
```

## 自定义自动装配
Spring Boot 提供了自动装配的功能，我们也可以自定义自动装配。

自定义自动装配的步骤如下：
1. 创建一个starter模块，只保留`pom.xml文件`
2. 创建一个autoconfiguration模块，该模块中包含需要自动装配的类，并且在starter中引入该模块
3. 在autoconfiguration模块中定义自动装配的功能，使用`@Configuration`注解标记需要自动装配的类，使用`@ConditionalOnXXX`注解标记需要自动装配的类，使用`@Bean`注解标记需要自动装配的方法，最后在`META-INF/org.springframework.boot.autoconfigure.AutoConfiguration.imports`文件中添加自动装配的类

注意事项：在autoconfiguration模块中，遇到需要`@Autowired`自动注入对象时，可以使用`@EnableConfigurationProperties`注解，将需要自动注入的对象注入到容器中。
```java
@Configuration
@EnableConfigurationProperties(MybatisProperties.class)
public class MybatisAutoConfiguration {
    @Autowired
    private MybatisProperties properties;
}
```
`@EnableConfigurationProperties`注解包含了@Import注解，该注解会将需要自动注入的对象注入到容器中。