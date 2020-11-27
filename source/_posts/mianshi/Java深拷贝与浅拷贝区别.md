---
title: Java深拷贝与浅拷贝区别
date: 2020-11-27 16:41:53
categories: 
- 面试
tags:
- java理论复习
---

### 为什么要使用数据拷贝

### 场景
在开发中经常会遇到，从父级类中拷贝属性到子类。一般使用2种方法：
- 一个一个的get/set
- 使用BeanUtils.copyProperties
显然BeanUtils更加美观

### BeanUtils是深拷贝还是浅拷贝呢？

- 知识铺垫
    - Java中的数据类型分为基本数据类型和引用数据类型。对于二者，在进行赋值时，用作方法参数或返回时，会有值传递和引用传递的差别

- 浅拷贝
    - 基本数据类型：浅拷贝会直接进行值传递，也就是将属性值复制一份给新的对象。
    - 引用数据类型：如果成员变量时某个对象或数组，那么浅拷贝会进行引用传递，也就是将成员变量引用值（内存地址）复制一份给新的对象
- 浅拷贝的实现方式：通过构造函数参数为该类，可以完成浅拷贝
```java
public class Test {

    public static void main(String[] args) {
        CopyTest copyTest = new CopyTest("a", "b");
        CopyTest copyTest1 = new CopyTest();
        BeanUtils.copyProperties(copyTest, copyTest);
        copyTest1.setA("A");
        System.out.println(copyTest1.getA() == copyTest.getA());
        CopyTest copyTest2 = new CopyTest(copyTest);
        copyTest2.setB("B");
        System.out.println(copyTest2.getB()==copyTest2.getB());
    }

    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    static class CopyTest {
        private String a;
        private String b;

        public CopyTest(CopyTest copyTest) {

        }
    }
}
```
 
 - 运行结果
 
 ```text
false
true
```

- 深拷贝
    - 

- 结果分析








