---
layout: post
title: Mybatis工作流程
tags: Mybatis
author: neal

---
* content
{:toc}


## Mybatis 流程

* 读取配置文件
* 创建SqlSessionFactoryBuilder对象
* 根据SqlSessionFactoryBuilder创建SqlSessionFactory对象
* 通过SqlSessionFactory创建SqlSession
* 为Dao接口生成代理类
* 调用接口访问数据库

