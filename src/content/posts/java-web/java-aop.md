---
title: springboot-aop
published: 2025-03-26
description: 'aop的使用'
image: ''
tags: [aop]
category: 'java-web'
draft: false 
---
### Spring Boot中的AOP详解

#### 一、AOP核心概念与原理
**AOP（Aspect-Oriented Programming）** 是一种编程范式，用于处理横跨多个模块的横切关注点（如日志、事务、安全等），通过解耦增强逻辑与业务代码，提升可维护性。

##### 核心概念：
1. **切面（Aspect）**：封装横切逻辑的模块（如日志切面）。
2. **连接点（Join Point）**：程序执行中的特定点（如方法调用、异常抛出）。
3. **通知（Advice）**：切面在连接点执行的动作，分为五种类型。
4. **切点（Pointcut）**：通过表达式匹配连接点，确定通知的应用位置。
5. **目标对象（Target）**：被代理的原始对象。
6. **代理（Proxy）**：由AOP框架生成的增强对象，拦截方法调用。

##### 实现原理：
- **动态代理**：Spring AOP通过动态代理实现：
  - **JDK动态代理**：基于接口，要求目标类实现至少一个接口。
  - **CGLIB代理**：基于子类继承，代理类继承目标类（适用于无接口的类）。
- **代理选择**：默认优先使用JDK代理，若无接口则用CGLIB。可通过配置强制使用CGLIB：
  ```properties
  spring.aop.proxy-target-class=true
  ```

---

#### 二、Spring Boot中使用AOP
##### 1. 引入依赖
在`pom.xml`中添加：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

##### 2. 定义切面
使用`@Aspect`和`@Component`注解定义切面：
```java
@Aspect
@Component
public class LoggingAspect {
    // 切点与通知定义
}
```

##### 3. 定义切点（Pointcut）
通过`@Pointcut`和表达式匹配目标方法：
```java
@Pointcut("execution(* com.example.service.*.*(..))")
public void serviceMethods() {}
```

**切点表达式语法**：
- `execution(<修饰符> <返回类型> <类路径>.<方法名>(<参数>) <异常>)`
  - 示例：`execution(public * com.example.service.*.*(..))` 匹配所有service包下的public方法。

##### 4. 定义通知（Advice）
- **@Before**：目标方法执行前执行。
- **@After**：方法执行后执行（无论是否异常）。
- **@AfterReturning**：方法成功返回后执行。
- **@AfterThrowing**：方法抛出异常后执行。
- **@Around**：包裹目标方法，控制其执行（需手动调用`proceed()`）。

**示例**：
```java
@Before("serviceMethods()")
public void logBefore(JoinPoint joinPoint) {
    String methodName = joinPoint.getSignature().getName();
    System.out.println("Method " + methodName + " started.");
}

@Around("serviceMethods()")
public Object logAround(ProceedingJoinPoint joinPoint) throws Throwable {
    long start = System.currentTimeMillis();
    Object result = joinPoint.proceed(); // 必须调用proceed()
    long duration = System.currentTimeMillis() - start;
    System.out.println("Method executed in " + duration + "ms");
    return result;
}
```

---

#### 三、高级用法与配置
##### 1. 切面执行顺序
通过`@Order`注解控制多个切面的执行顺序（值越小优先级越高）：
```java
@Aspect
@Component
@Order(1)
public class LoggingAspect { /* ... */ }

@Aspect
@Component
@Order(2)
public class SecurityAspect { /* ... */ }
```

##### 2. 自定义注解实现AOP
定义注解：
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Loggable {}
```

切面中使用：
```java
@Pointcut("@annotation(com.example.annotation.Loggable)")
public void loggableMethods() {}

@Before("loggableMethods()")
public void logCustomAnnotation(JoinPoint joinPoint) {
    // 处理带有@Loggable注解的方法
}
```

---

#### 四、获取代理方法的详细信息
在Spring AOP中，通过`JoinPoint`对象可以获取代理方法的详细信息，包括方法签名、参数、目标对象和注解等。以下是具体实现方式及示例：

---

##### **1. 获取方法基本信息**
通过`JoinPoint`的`getSignature()`方法获取`MethodSignature`，进而获取方法元数据：

```java
@Before("execution(* com.example.service.*.*(..))")
public void logMethodInfo(JoinPoint joinPoint) {
    // 获取方法签名
    MethodSignature signature = (MethodSignature) joinPoint.getSignature();
    
    // 方法名
    String methodName = signature.getName();
    
    // 方法返回类型
    Class<?> returnType = signature.getReturnType();
    
    // 方法参数类型列表
    Class<?>[] parameterTypes = signature.getParameterTypes();
    
    // 方法参数值列表
    Object[] args = joinPoint.getArgs();
    
    System.out.println("Method: " + methodName);
    System.out.println("Return Type: " + returnType);
    System.out.println("Args: " + Arrays.toString(args));
}
```

---

##### **2. 获取目标对象和代理对象**
- `getTarget()`：获取被代理的原始目标对象。
- `getThis()`：获取代理对象本身。

```java
@Before("execution(* com.example.service.*.*(..))")
public void logTargetAndProxy(JoinPoint joinPoint) {
    // 目标对象（原始对象）
    Object target = joinPoint.getTarget();
    System.out.println("Target Class: " + target.getClass().getName());

    // 代理对象
    Object proxy = joinPoint.getThis();
    System.out.println("Proxy Class: " + proxy.getClass().getName());
}
```

---

##### **3. 获取方法上的注解**
使用反射获取方法上的注解，建议通过`AnnotationUtils`处理注解继承问题：

```java
@Before("@annotation(com.example.Loggable)")
public void logAnnotation(JoinPoint joinPoint) {
    MethodSignature signature = (MethodSignature) joinPoint.getSignature();
    Method method = signature.getMethod();
    
    // 获取方法上的注解
    Loggable loggable = AnnotationUtils.findAnnotation(method, Loggable.class);
    if (loggable != null) {
        System.out.println("Log Level: " + loggable.level());
    }
}
```

---

##### **4. 获取实际目标类的方法**
若目标方法来自接口，需通过目标对象获取实现类的方法：

```java
@Before("execution(* com.example.service.*.*(..))")
public void logRealMethod(JoinPoint joinPoint) throws NoSuchMethodException {
    MethodSignature signature = (MethodSignature) joinPoint.getSignature();
    Object target = joinPoint.getTarget();
    
    // 获取目标类的方法（而非接口方法）
    Method realMethod = target.getClass().getMethod(
        signature.getName(), 
        signature.getParameterTypes()
    );
    
    System.out.println("Real Method Class: " + realMethod.getDeclaringClass());
}
```

---

###### **5. 判断代理类型**
使用Spring工具类`AopUtils`判断代理是JDK动态代理还是CGLIB：

```java
import org.springframework.aop.support.AopUtils;

@Before("execution(* com.example.service.*.*(..))")
public void logProxyType(JoinPoint joinPoint) {
    Object proxy = joinPoint.getThis();
    
    if (AopUtils.isJdkDynamicProxy(proxy)) {
        System.out.println("Proxy Type: JDK Dynamic Proxy");
    } else if (AopUtils.isCglibProxy(proxy)) {
        System.out.println("Proxy Type: CGLIB Proxy");
    }
}
```

---

##### **6. 获取方法参数的名称（需启用调试信息）**
若需获取参数名称，需在编译时保留调试信息（`-parameters`编译选项）：

```java
@Before("execution(* com.example.service.*.*(..))")
public void logParameterNames(JoinPoint joinPoint) {
    MethodSignature signature = (MethodSignature) joinPoint.getSignature();
    String[] paramNames = signature.getParameterNames();
    Object[] paramValues = joinPoint.getArgs();
    
    for (int i = 0; i < paramNames.length; i++) {
        System.out.println(paramNames[i] + ": " + paramValues[i]);
    }
}
```

---

##### **注意事项**
1. **内部方法调用**：  
   同一类中的内部方法调用（如`this.internalMethod()`）不会触发AOP代理，需通过代理对象调用：
   ```java
   ((MyService) AopContext.currentProxy()).internalMethod();
   ```
   并配置`@EnableAspectJAutoProxy(exposeProxy = true)`。

2. **目标方法可见性**：  
   Spring AOP只能拦截`public`方法，私有方法无法被代理。

3. **注解继承问题**：  
   若注解在接口方法上，直接通过`method.getAnnotation()`可能无法获取，需使用`AnnotationUtils.findAnnotation()`。

4. **代理类型差异**：  
   - JDK代理：`getTarget()`返回目标对象，`getThis()`返回代理对象。  
   - CGLIB代理：`getTarget()`可能返回原始对象，`getThis()`返回代理子类。

---

##### **完整示例**
```java
@Aspect
@Component
public class MethodInfoAspect {

    @Before("execution(* com.example.service.*.*(..))")
    public void logFullMethodDetails(JoinPoint joinPoint) {
        // 方法签名
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();

        // 基本信息
        String methodName = method.getName();
        Class<?> returnType = method.getReturnType();
        Object[] args = joinPoint.getArgs();

        // 目标对象和代理对象
        Object target = joinPoint.getTarget();
        Object proxy = joinPoint.getThis();

        // 打印信息
        System.out.println("Method: " + methodName);
        System.out.println("Return Type: " + returnType);
        System.out.println("Args: " + Arrays.toString(args));
        System.out.println("Target Class: " + target.getClass().getName());
        System.out.println("Proxy Class: " + proxy.getClass().getName());

        // 判断代理类型
        if (AopUtils.isJdkDynamicProxy(proxy)) {
            System.out.println("Proxy Type: JDK");
        } else if (AopUtils.isCglibProxy(proxy)) {
            System.out.println("Proxy Type: CGLIB");
        }

        // 获取方法注解
        Loggable loggable = AnnotationUtils.findAnnotation(method, Loggable.class);
        if (loggable != null) {
            System.out.println("Loggable Level: " + loggable.level());
        }
    }
}
```

---

通过以上方法，可以全面获取代理方法的相关信息，适用于日志记录、权限检查、性能监控等场景。注意处理代理类型和注解继承等细节问题。

#### 四、注意事项与常见问题
1. **方法可见性限制**：
   - Spring AOP只能拦截**public**方法，私有方法无法被代理。

2. **内部方法调用问题**：
   - 同一类中的方法内部调用（如`this.internalMethod()`）不会触发AOP，需通过代理对象调用：
     ```java
     ((MyService) AopContext.currentProxy()).internalMethod();
     ```
   - 需启用`@EnableAspectJAutoProxy(exposeProxy = true)`。

3. **代理方式选择**：
   - CGLIB代理时，目标类不能为`final`，方法不能为`final/static`。

4. **性能考量**：
   - 避免在频繁调用的方法上使用复杂切面逻辑，尤其是`@Around`。

5. **异常处理**：
   - `@AfterThrowing`仅在方法抛出异常时触发，需注意异常是否被其他通知处理。

6. **精确切点表达式**：
   - 避免过于宽泛的表达式（如`execution(* *(..))`），可能拦截到不需要的方法。

---

#### 五、示例：完整日志切面
```java
@Aspect
@Component
public class LoggingAspect {
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceLayer() {}

    @Before("serviceLayer()")
    public void logMethodEntry(JoinPoint joinPoint) {
        String methodName = joinPoint.getSignature().toShortString();
        Object[] args = joinPoint.getArgs();
        System.out.println("Entering " + methodName + " with args: " + Arrays.toString(args));
    }

    @AfterReturning(pointcut = "serviceLayer()", returning = "result")
    public void logMethodExit(JoinPoint joinPoint, Object result) {
        System.out.println("Exiting " + joinPoint.getSignature().toShortString() + " with result: " + result);
    }

    @AfterThrowing(pointcut = "serviceLayer()", throwing = "ex")
    public void logException(JoinPoint joinPoint, Exception ex) {
        System.err.println("Exception in " + joinPoint.getSignature().toShortString() + ": " + ex.getMessage());
    }
}
```

---

#### 六、最佳实践
1. **按模块拆分切面**：如日志、事务、缓存切面分离。
2. **复用切点表达式**：通过`@Pointcut`集中管理。
3. **结合自定义注解**：提高代码可读性和灵活性。
4. **测试验证**：通过单元测试确认切面生效，避免拦截遗漏或误拦截。