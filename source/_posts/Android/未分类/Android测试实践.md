---
title: Android测试实践 
tags: Android 
toc: true
---

## **为什么要做测试？**

很多公司为了节省`成本`，基本上是不会要求开发写单元测试的。自己做完的功能，动手点点看看基本流程是否通畅就丢给测试了。
项目小的时候这样也不会有啥问题。随着项目的复杂度越来越高，人员越来越多，不同的人员改动到同一块代码，但彼此对原来的逻辑不是很清楚（因为代码量实在太多，想要一个人弄清楚所有细节已经不太可能）。
这也是为什么大家宁肯写新的代码也不愿意改旧的代码的原因。而如果每个功能在开发的同时配置了单元测试，那么理论上新的功能 开发完之后，所有的单元测试应该都是可以跑通的。这样就能自动帮我们检查出来一些难以发现的bug。

## Android如何进行单元测试

- androidTest目录：应包含在实际或虚拟设备上运行的测试。此类测试包括集成测试，端到端测试以及仅JVM无法验证应用程序功能的其他测试。
- test目录：应包含在本地计算机上运行的测试，例如单元测试

## 测试常用库

- JUnit
- Mockito
- PowerMock
- Espresso

## JUnit

TestCase TestResult

TestSuite 测试套件

在实际项目中，随着项目进度的开展，单元测试类会越来越多，可是直到现在我们还只会一个一个的单独运行测试类，这在实际项目实践中肯定是不可行的。 为了解决这个问题，JUnit 提供了一种批量运行测试类的方法，叫做测试套件。

断言： Assert

标注：

标注|描述
|---|---|
@Test|该方法是一个测试用例
@Before|每个测试方法执行之前执行
@After|每个测试方法执行后执行
@BeforeClass|一个测试文件只执行一次
@AfterClass|一个测试文件最后执行一次
@Ignore|这个注解是用来忽略有关不需要执行的测试的。

## Mockito

模拟被测对象依赖的对象

### 参考

- [https://dunwu.github.io/javatech/test/mockito.html](https://dunwu.github.io/javatech/test/mockito.html)

## PowerMock

模拟被测对象依赖的静态方法

## Robolectric

Android SDK测试框架

### Config配置

### 参考

- [https://maxwell-nc.github.io/android/robolectricTest.html](https://maxwell-nc.github.io/android/robolectricTest.html)
- [https://www.jianshu.com/p/6c23f2681105](https://www.jianshu.com/p/6c23f2681105)

## Espresso
