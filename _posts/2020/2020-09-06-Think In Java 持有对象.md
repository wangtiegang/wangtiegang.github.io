---
layout:     post
title:      "Think In Java 持有对象"
subtitle:   "《Think In Java》阅读笔记"
date:       2020-09-06 18:58:00
author:     "wangtiegang"
header-img: ""
catalog: false
tags:
    - java
---

通常，程序总是运行时才知道要创建一组对象，在此之前不知道要创建多少数量，甚至不知道对象的类型。为了解决这个普遍的问题，就不能依靠创建命名的方式来创建并持有每一个对象。

```java
MyType mt1;
MyType mt2;
// ...
// 不知道要创建多少个
```

Java 有多种方式解决这个问题，比如数组。数组是保存一组对象最有效的方式，但是数组具有固定的尺寸，无法满足不知道数量的情况。因此 Java 提供了一套相当完整的容器类来解决这个问题，其中基本类型是 List，Set，Queue，Map。这些容器都可以自动的调整自己的尺寸。

#### List

List 承诺可以将元素维护在特定的序列中，List 在 Collection 的基础上添加了大量的方法，有两种常见的类型的 List。

* ArrayList 擅长随机访问元素，但是在中间插入和移除元素时较慢
* LinkedList 擅长快速的在中间进行插入和删除元素，但是在随机访问方面较慢。同时内部还添加了可以使其用作栈，队列或双端队列的方法

> 向上转型

需要注意的是，平时我们使用 List，通常如下定义

```java
List list = new LinkedList();
```

此时 LinkedList 已经向上转型为 List，通常这样是因为面向接口编程，降低程序的耦合，但这也意味着后续使用对象对时候丢失了 LinkedList 在 List 中未包含的方法，如果想使用这些方法，就不能将它们向上转型为更通用的接口，同理 TreeMap 也具有 Map 接口中未包含的方法。

> Arrays.asList

可以将 ```Arrays.asList()``` 的输出当作 List，但是这种情况下，它的底层是数组，不能再调整尺寸。

> equals

当对比一个元素时，比如查找元素，删除元素，都会使用到 ```equals()``` 方法，因此必须意识到 List 的行为根据 ```equals()``` 的行为而有所变化。


#### Set

Set 不保存重复的元素，正因如此，查找就成了 Set 中最重要的操作，因此通常会选择 HashSet 的实现，它专门对查找进行了优化。

HashSet 是无序的，如果想对结果排序，一种方式是使用 TreeSet 来代替 HashSet。

#### Queue

LinkedList 提供了方法以支持队列的行为，并且实现了 Queue 接口，因此 LinkedList 是 Queue 的一种实现，可以向上转型为 Queue。

#### 其他

新程序中不应该使用已经过时的 Vector，Hashtable，Stack 对象。