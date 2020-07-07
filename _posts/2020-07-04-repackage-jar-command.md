---
layout: post
title: 替换jar包class文件
tags: linux
author: neal

---
* content
{:toc}
### **linux重新打包替换class文件**



```shell
# 1. 查找class文件位置
jar -tvf xxx.jar | grep xxx.class
# 2. 单独解压指定class文件
jar -xvf xxx.jar BOOT-INF/classes/xxx/xxx.class
# 3. 将要替换的class文件打包进jar
jar -uvf xxx.jar BOOT-INF/classes/xxx/xxx.class
```



引用： https://blog.csdn.net/yk614294861/article/details/106220707/