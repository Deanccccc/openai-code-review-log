### 严重问题 (必须修复)
[运行时异常 / 测试设计缺陷]：在 `test02` 方法中直接进行整数除以零（`a / b`，其中 `b = 0`），会抛出 `ArithmeticException`，导致测试崩溃失败。  
```java
@Test
public void test02() {
    int a = 1;
    int b = 0;
    System.out.println(a / b);  // 这里会抛出异常
}
```
**建议**：  
1. 如果本意是测试除零场景下的异常抛出，应使用 `assertThrows` 来明确验证行为，而不是直接 `println`。  
2. 如果并非测试异常，则需确保除数不为零，并为方法添加有意义的业务断言。  

**修复示例**（测试异常场景）：
```java
@Test
public void testDivideByZeroShouldThrowException() {
    int a = 1;
    int b = 0;
    assertThrows(ArithmeticException.class, () -> {
        int result = a / b;
    });
}
```
**修复示例**（常规除法测试）：
```java
@Test
public void testDivision() {
    int a = 10;
    int b = 2;
    assertEquals(5, a / b);
}
```

### 🟡 建议改进 (非阻塞，但建议优化)
[测试规范]：该测试方法没有任何断言（`assertXXX`），仅使用 `System.out.println`，无法自动验证结果，违背单元测试“自校验”原则。即便是用于探索性调试，也不应提交到代码仓库。  
**建议**：移除 `println`，并添加明确的断言来验证预期结果。如果确实需要输出中间值，可在测试中使用日志框架而非标准输出。

[代码命名与职责]：方法名 `test02` 缺乏描述性，无法体现测试意图。  
**建议**：采用如 `shouldThrowExceptionWhenDividingByZero` 或 `testDivisionScenario` 等更具可读性的命名。