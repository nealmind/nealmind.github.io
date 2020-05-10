---
layout: post
title: Spring源码BeanFactory和ApplicationContext区别
tags: Spring
author: neal
---
* content
{:toc}
## BeanFactory和ApplicationContext区别：

ApplicationContext是BeanFactory的子接口，对BeanFactory进行了一定的扩展，增加了：

* MessageSource，提供国际化的消息访问
* 继承了ResourceLoader，支持资源访问
* 提供了Bean的自动装配
* 提供了各种应用层的Context实现

例如，构建BeanFactory时，不会实例化bean，只有调用其getBean方法时，才会对bean进行实例化；

而构建ApplicationContext时，会调用AbstractApplicationContext的refresh方法进行BeanDefinition的注册和bean的实例化；其中，BeanDefinition的注册是由两个子类AbstractRefreshableApplication和AnnotationConfigApplicationContext实现的，分别对应xml的加载和基于注解的加载；xml是在刷新方法中进行BeanDefinition解析，基于注解的是在调用刷新方法之前解析；

**BeanFactory和ApplicationContext的区别在于：**

对于singleton的bean，BeanFactory只有在使用bean时（调用getBean）才会进行实例化，节省内存，类似于懒加载；

ApplicationContext相反，是在构建容器后就进行实例化，优点是加载迅速；