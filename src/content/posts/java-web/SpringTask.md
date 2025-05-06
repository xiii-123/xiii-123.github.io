---
title: Spring Task 定时任务
published: 2025-04-27
description: 'Spring Task 定时任务'
image: ''
tags: [定时任务, Spring]
category: 'java-web'
draft: false 
---

Spring Task 是 Spring 框架提供的轻量级定时任务调度工具，用于简化定时任务的开发。相比 Quartz 等复杂调度框架，Spring Task 更轻量、配置简单，适合单机环境下的周期性任务需求。以下是其核心功能和使用细节：

### **一、Spring Task 的核心功能**
1. **基于注解的定时任务**  
   通过 `@Scheduled` 注解快速定义任务的执行时间规则。
2. **支持多种触发策略**  
   如固定频率（`fixedRate`）、固定延迟（`fixedDelay`）、Cron 表达式等。
3. **集成 Spring 容器**  
   可直接使用 Spring 的依赖注入（`@Autowired`）和其他特性。
4. **动态任务调整**  
   结合 `ScheduledTaskRegistrar` 可实现动态注册或取消任务。

---

### **二、快速使用步骤**
#### 1. 启用 Spring Task
在 Spring Boot 配置类添加 `@EnableScheduling`：
```java
@SpringBootApplication
@EnableScheduling // 启用定时任务
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```

#### 2. 定义定时任务
使用 `@Scheduled` 注解标记方法：
```java
@Component
public class MyTask {

    // 每5秒执行一次（固定频率）
    @Scheduled(fixedRate = 5000)
    public void task1() {
        System.out.println("Task1 executed at: " + new Date());
    }

    // 使用 Cron 表达式（每天12点执行）
    @Scheduled(cron = "0 0 12 * * ?")
    public void task2() {
        System.out.println("Task2 executed at: " + new Date());
    }
}
```

---

### **三、@Scheduled 参数详解**
1. **`fixedRate`**  
   - 固定频率执行（单位：毫秒），无论前一次任务是否完成。
   - 示例：`@Scheduled(fixedRate = 5000)`。
   - **注意**：若任务执行时间超过间隔，后续任务会立即触发。

2. **`fixedDelay`**  
   - 固定延迟执行（单位：毫秒），等待前一次任务完成后，再延迟指定时间执行。
   - 示例：`@Scheduled(fixedDelay = 5000)`。

3. **`cron`**  
   - 通过 Cron 表达式定义复杂时间规则。
   - 示例：`@Scheduled(cron = "0 0/30 9-17 * * MON-FRI")`（工作日的9点到17点每30分钟执行）。
   - **Cron 格式**：`秒 分 时 日 月 周 年（可选）`。

4. **`initialDelay`**  
   - 首次任务执行的延迟时间（需与 `fixedRate` 或 `fixedDelay` 配合使用）。
   - 示例：`@Scheduled(initialDelay = 1000, fixedRate = 5000)`。

---

### **四、线程池配置**
默认情况下，Spring Task 使用单线程执行任务，可能导致任务阻塞。需自定义线程池：

#### 方案1：配置 `TaskScheduler` Bean
```java
@Configuration
public class SchedulerConfig {

    @Bean
    public TaskScheduler taskScheduler() {
        ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
        scheduler.setPoolSize(10); // 线程池大小
        scheduler.setThreadNamePrefix("scheduled-task-");
        scheduler.setAwaitTerminationSeconds(60);
        scheduler.setWaitForTasksToCompleteOnShutdown(true);
        return scheduler;
    }
}
```

#### 方案2：实现 `SchedulingConfigurer`
```java
@Configuration
public class SchedulerConfig implements SchedulingConfigurer {

    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        taskRegistrar.setScheduler(Executors.newScheduledThreadPool(10));
    }
}
```

---

### **五、动态任务管理**
通过 `ScheduledTaskRegistrar` 动态注册/取消任务：
```java
@Component
public class DynamicTask implements SchedulingConfigurer {

    private ScheduledTaskRegistrar taskRegistrar;
    private ScheduledFuture<?> future;

    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        this.taskRegistrar = taskRegistrar;
    }

    // 动态注册任务
    public void startTask() {
        future = taskRegistrar.getScheduler().schedule(
            () -> System.out.println("Dynamic Task"),
            new CronTrigger("0/10 * * * * ?")
        );
    }

    // 取消任务
    public void stopTask() {
        if (future != null) {
            future.cancel(true);
        }
    }
}
```

---

### **六、注意事项**
1. **避免长时间阻塞任务**  
   默认单线程下，长时间任务会导致后续任务延迟。需配置线程池或异步执行（结合 `@Async`）。

2. **Cron 表达式时区问题**  
   默认使用服务器时区，可通过 `zone` 属性指定：  
   `@Scheduled(cron = "0 0 12 * * ?", zone = "Asia/Shanghai")`
   可以使用在线工具生成 Cron 表达式，如 [CronMaker](http://www.cronmaker.com/)。

3. **异常处理**  
   任务方法内需自行捕获异常，否则会导致任务终止。

4. **分布式环境问题**  
   Spring Task 本身不支持分布式调度，需结合分布式锁（如 Redis 锁）或改用 Quartz 集群。

---

### **七、适用场景**
- **简单单机任务**：如数据统计、缓存刷新、日志清理等。
- **轻量级需求**：无需复杂调度策略（如任务依赖、分片等）。
- **快速开发**：避免引入 Quartz 等重型框架的复杂度。
