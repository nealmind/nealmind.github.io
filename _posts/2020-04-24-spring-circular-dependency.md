---
layout: post
title: Spring循环依赖
tags: Spring
author: neal

---
* content
{:toc}
## spring循环依赖

### setter依赖

通过三级缓存解决

* **singletonObjects** 缓存创建完成的bean
* **earlySingletonObjects** 缓存正在创建中的bean
* **singletonFactories** 缓存通过工厂创建的bean

### 构造器依赖

* **重新设计依赖关系，避免构造器循环依赖**
* 通过使用@Lazy注解进行懒加载
* 使用@PostConstruct注解，在构造函数被调用后添加依赖关系，缓存到

三级缓存中，从而解决循环依赖

* 实现InitializingBean 接口，在afterPropertiesSet方法中从容器中获取bean并添加依赖关系

  