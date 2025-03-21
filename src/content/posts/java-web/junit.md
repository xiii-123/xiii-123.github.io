---
title: Junit
published: 2025-03-17
description: 'Junit常用断言和注解'
image: ''
tags: [Junit]
category: 'java-web'
draft: false 
---

JUnit是一个开源的Java单元测试框架，主要用于编写和运行测试代码。它提供了丰富的注解、断言以及测试运行器，使得开发者可以轻松地编写测试用例，验证代码的正确性。JUnit的核心思想是“先测试，后编码”，鼓励开发者采用测试驱动开发（TDD）的模式。

## JUnit中常见的断言
断言是测试代码中用来验证预期结果与实际结果是否一致的关键方法。JUnit提供了多种断言方法，以下是一些常见的断言：
1. `assertEquals(expected, actual)`：验证预期值与实际值是否相等。
2. `assertTrue(condition)`：验证条件是否为真。
3. `assertFalse(condition)`：验证条件是否为假。
4. `assertNotNull(object)`：验证对象是否不为null。
5. `assertNull(object)`：验证对象是否为null。
6. `assertSame(expected, actual)`：验证两个对象是否引用同一实例。
7. `assertNotSame(expected, actual)`：验证两个对象是否不引用同一实例。
8. `assertArrayEquals(expectedArray, actualArray)`：验证两个数组是否相等。
## JUnit中常见的注解
注解是JUnit中用于标记测试方法、测试类以及配置测试行为的特殊标记。以下是一些常见的注解：
1. `@Test`：标记一个方法为测试方法。
2. `@Before`：标记一个方法在每一个测试方法执行之前执行。
3. `@After`：标记一个方法在每一个测试方法执行之后执行。
4. `@BeforeClass`：标记一个静态方法在所有测试方法执行之前执行一次。
5. `@AfterClass`：标记一个静态方法在所有测试方法执行之后执行一次。
6. `@Ignore`：标记一个测试方法被忽略，不执行。
7. `@RunWith`：指定测试运行器。
8. `@ParameterizedTest`：用于参数化测试，指定测试数据，结合`@ValueSource`使用。
9. `@ValueSource`注解可以用于提供一系列的值作为参数化测试的输入。
## 代码示例
以下是一个全面的JUnit最佳实践代码示例，包括测试类、测试方法、断言、注解以及参数化测试：
```java
import org.junit.*;
import org.junit.runner.RunWith;
import org.junit.runners.Parameterized;
import java.util.Arrays;
import java.util.Collection;
@RunWith(Parameterized.class)
public class CalculatorTest {
    private int number1;
    private int number2;
    private int expected;
    public CalculatorTest(int number1, int number2, int expected) {
        this.number1 = number1;
        this.number2 = number2;
        this.expected = expected;
    }
    @BeforeClass
    public static void setUpBeforeClass() {
        // 在所有测试方法执行之前执行一次
        System.out.println("BeforeClass");
    }
    @AfterClass
    public static void tearDownAfterClass() {
        // 在所有测试方法执行之后执行一次
        System.out.println("AfterClass");
    }
    @Before
    public void setUp() {
        // 在每一个测试方法执行之前执行
        System.out.println("Before");
    }
    @After
    public void tearDown() {
        // 在每一个测试方法执行之后执行
        System.out.println("After");
    }
    @Test
    public void testAdd() {
        Calculator calculator = new Calculator();
        int result = calculator.add(number1, number2);
        assertEquals(expected, result);
    }
    @Test
    @Ignore("暂时忽略这个测试方法")
    public void testSubtract() {
        Calculator calculator = new Calculator();
        int result = calculator.subtract(number1, number2);
        assertEquals(expected, result);
    }
    @ParameterizedTest
    @ValueSource(ints = {1, 2, 3, 4, 5})
    void testIsPositive(int number) {
        assertTrue(calculator.isPositive(number), "The number should be positive");
    }
    // 方法实现省略
}
```