---
layout: post
title: 线程中断
tags: java基础
author: neal

---
* content
{:toc}

线程中断主要依赖Thread类的三个方法：
`interrupt()`:中断调用的线程，如果是阻塞在Object的join(),wait(),sleep()
等方法上，抛出InterruptedException异常并清除中断状态；
如果是阻塞在io相关可中断方法上，不会清除中断状态，会抛出对应异常；
`interrupted()`:静态方法，返回当前线程中断状态，并清除状态，也就是如果调用两次，第二次必定返回false；
`isInteruppted()`: 返回调用线程的中断状态，不进行任何处理；和`interrupted()`方法一样都是调用了native方法：
`isInterrupted(boolean ClearInterrupted)`，区别是前者使用了当前线程，并且ClearInterrupted 设置为true，即
清除中断状态，而此方法应用于调用线程，并且没有清除中断状态。
