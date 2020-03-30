---
layout: post
title: JVM常用参数
tags: JVM
author: neal

---
* content
{:toc}
`-Xms:` 堆内存最小值

`-Xmx:` 堆内存最大值（一般和`-Xms`相等）

`-Xss:` 每个线程对应栈内存大小

`-XX:MaxMetaspaceSize:` 元数据内存大小

`-XX:SurvivorRatio:` eden/survivor大小比例，默认为8

`-XX:MaxTenuringThreshold:` 进入老年代的MinorGC次数临界值，默认为15

`-XX:PretenureSizeThreshold:` 大对象阈值，超过这个阈值的直接分配到老年代，默认为0

ParNew + CMS收集器