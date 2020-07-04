---
layout: post
title: 替换jar包class文件
tags: linux
author: neal

---
* content
{:toc}
### **linux重新打包替换class文件**



1. 查找所要替换的文件目录：

```shell
# jar -tvf xxx.jar |grep DateUtil.class
##返回class路径
BOOT-INF/classes/cn/xxx/utils/DateUtil.class
```

2. 将查找到的class文件单独解压

```shell
# jar -xvf xxx.jar BOOT-INF/classes/cn/xxx/utils/DateUtil.class
```

3. 上传替换的class文件，之后重新打包进jar

```shell
# jar -uvf xxx.jar BOOT-INF/classes/cn/xxx/utils/DateUtil.class
```





引用： https://blog.csdn.net/yk614294861/article/details/106220707/