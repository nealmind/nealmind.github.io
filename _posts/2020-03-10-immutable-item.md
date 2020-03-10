---
layout: post
title: Immutable对象满足条件
tags: Java基础
author: neal

---
* content
{:toc}

## Immutable对象满足条件
* 对象创建后，其状态就不能修改
* 对象的所有域都是final修饰
* 对象是正确创建的(即在创建过程中没有出现`this`溢出)

Immutable对象是线程安全的，因为其状态不可变，并且只有构造函数控制；
---
《Java并发编程实战》

