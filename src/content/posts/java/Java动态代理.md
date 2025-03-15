---
title: Java动态代理
published: 2025-03-15
description: '太久没用Java，都忘了动态代理是什么'
image: ''
tags: [java, 动态代理]
category: 'java'
draft: false 
---
Java中的动态代理是一种在运行时创建一个实现了一组给定接口的新类的能力，并且可以在运行时确定这个类要实现的方法的行为。动态代理通常用于实现AOP（面向切面编程）和拦截器模式。
## 使用到的类：
1. `java.lang.reflect.Proxy`：这是创建动态代理实例的主要类。
2. `java.lang.reflect.InvocationHandler`：这是一个接口，由需要处理代理实例方法调用的类实现。
## 主要参数：
- `ClassLoader loader`：用于定义代理类的类加载器。
- `Class<?>[] interfaces`：代理类要实现的接口列表。
- `InvocationHandler h`：当代理实例上的方法被调用时，会调用处理器的`invoke`方法。
## 注意事项：
1. 动态代理只能为接口创建代理实例，不能为类创建代理。
2. 所有被代理的方法都必须在某个接口中声明，否则代理类不会暴露这些方法。
3. 当调用代理对象的方法时，实际调用的是`InvocationHandler`的`invoke`方法。
4. 代理类是在运行时由Java反射API动态生成的，因此它们不会出现在源代码中。
## 最佳实践示例：
以下是一个简单的动态代理示例，其中创建了接口`Star`，实现类`BigStar`以及代理类`StarProxy`。
```java
public interface Star {
    String sing(String name);

    String dance(String name);
}
```
```java
public class BigStar implements Star{
    private String name;

    public BigStar(String name) {
        this.name = name;
    }

    @Override
    public String sing(String name) {
        System.out.println(this.name+"正在唱"+name);
        return "谢谢";
    }

    @Override
    public String dance(String name) {
        System.out.println(this.name+"正在跳"+name);
        return "爱你们";
    }
}
```
```java
public class StarProxy {
    public static Star newProxy(Star bigStar){
        Star star = (Star) Proxy.newProxyInstance(
                StarProxy.class.getClassLoader(),
                new Class[]{Star.class},
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        if ("sing".equals(method.getName())){
                            System.out.println("准备话筒");
                        }else{
                            System.out.println("准备场地");
                        }
                        return method.invoke(bigStar,args);
                    }
                }
        );
        return star;
    }
}
```
```java
public class Main {
    public static void main(String[] args) {
        BigStar bigStar = new BigStar("周杰伦");
        Star proxy = StarProxy.newProxy(bigStar);
        System.out.println(proxy.sing("青花瓷"));
        System.out.println(proxy.dance("本草纲目"));
    }
}
```
结果：
```bash
准备话筒
周杰伦正在唱青花瓷
谢谢
准备场地
周杰伦正在跳本草纲目
爱你们
```