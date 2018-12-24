---
title: Java8 函数式
date: 2018-11-28T14:54:43.000Z
tags:
  - java
categories:
  - java
---

# Java8 函数式方程

### Java中重要的函数接口

接口               | 参数   | 返回类型
-------------------|--------|---------
Predicate<T\>      | T      | boolean
Consumer<T\>       | T      | viod
Function<T, R>     | T      | R
Supplier<T\>       | None   | T
UnaryOperator<T\>  | T      | T
BinaryOperator<T\> | (T, T) | T
<!-- more -->
### 常用的流操作

方法              | 作用                         | 示例
------------------|------------------------------|---------------------------------------------------------------------------------------------
collect(toList()) | 生成一个列表                 | Stream.of("a", "b", "c").collect(Collectors.toList());
map               | 将一种类型转换成另外一种类型 | Stream.of("a", "b", "c").map(s -> s.toUpperCase()). collect(Collectors.toList());
filter            | 过滤数据                     | Stream.of("a", "1", "b").filter(v -> isDigit(v.charAt(0))). collect(Collectors.toList());
flatMap           | 将多个Stream连接成一个Stream | Stream.of(asList(1, 2), asList(3, 4)).flatMap(n -> n.stream()).collect(Collectors.toList());
max/min           | 求最大和最小值               | Stream.of("a", "ab", "abc").max(Comparator.comparing(v -> v.length())).get());
reduce            | 累加操作                     | Steam.of(1, 2, 3).reduce(Integer::sum);

### 补充
 * [Java8 Optional](https://www.cnblogs.com/zhangboyu/p/7580262.html )
 * [Java8 @FunctionalInterface](https://www.cnblogs.com/runningTurtle/p/7092632.html)
 * [Stream 原理](https://www.cnblogs.com/Dorae/p/7779246.html)
