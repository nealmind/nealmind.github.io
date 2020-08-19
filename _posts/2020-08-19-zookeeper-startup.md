---
layout: post
title: Windows 安装配置ZooKeeper后台启动
tags: zk
author: neal

---
* content
{:toc}
## ZooKeeper下载

官网下载地址：http://zookeeper.apache.org/releases.html#download






## 安装

* 解压，并进入conf目录

* 复制 `zoo_sample.cfg` ，重命名为 `zoo.cfg`

* ss

* 编辑 `zoo.cfg` ，添加 `dataDir` 和 `dataLogDir` 

  ```shell
  # 注意是 双反斜杠 \\
  # 数据存储位置
  dataDir=D:\\zookeeper-3.4.11\\data
  # 日志存储位置
  dataLogDir=D:\\zookeeper-3.4.11\\log
  ```
  
## 服务配置
* 下载Apache出品的通用后台进程服务插件包：commons-daemon，下载地址：http://archive.apache.org/dist/commons/daemon/binaries/windows/ 通过镜像下载；
  
* 解压，复制`prunmgr.exe`、`prunsrv.exe` 到ZooKeepe安装路径的bin目录下, 64位系统则复制amd下的`prunsrv.exe`
  
* 在ZooKeeper根目录下新建安装脚本：`zk-service-install.bat`
  
  ```shell
  
  @echo off
   
  REM #
  REM # 说明：在Windows系统中安装ZooKeeper服务，实现以服务的方式启动ZooKeeper
  REM # 注意：此脚本必须拷贝到ZooKeeper的根目录，否则运行报错
  REM # ZK_HOME：			ZooKeeper安装目录
  REM # ZK_DATA_DIR：		ZooKeeper数据目录
  REM # ZK_SERVICE_NAME:	ZooKeeper服务名
  REM # DATE：			2018-8-10 15:37:55
  REM # Author：			许亮
  REM #
  REM # 服务启动命令：net start ZooKeeper
  REM # 服务停止命令：net stop ZooKeeper
   
  CD /d %~dp0
  SET ZK_HOME=%CD%
  SET ZK_DATA_DIR=%ZK_HOME%\data
  SET ZK_LOG_DIR=%ZK_HOME%\log
  SET ZK_SERVICE_NAME=ZooKeeper
  if not exist %ZK_DATA_DIR% mkdir %ZK_DATA_DIR%
  if not exist %ZK_LOG_DIR% mkdir %ZK_LOG_DIR%
   
  :: 安装ZooKeeper的Windows服务
  %ZK_HOME%\bin\prunsrv.exe "//IS//%ZK_SERVICE_NAME%" ^
  --DisplayName="%ZK_SERVICE_NAME%" ^
  --Description="%ZK_SERVICE_NAME%" ^
  --Startup=auto ^
  --StartMode=exe ^
  --StartPath=%ZK_HOME% ^
  --StartImage=%ZK_HOME%\bin\zkServer.cmd ^
  --StopPath=%ZK_HOME%\ ^
  --StopImage=%ZK_HOME%\zk-stop.bat ^
  --StopMode=exe ^
  --StopTimeout=5 ^
  --LogPath=%ZK_LOG_DIR% ^
  --LogPrefix=zookeeper-wrapper ^
  --PidFile=zookeeper.pid ^
  --LogLevel=Info ^
  --StdOutput=auto ^
  --StdError=auto
   
  pause
  ```
  
* 新建卸载服务脚本：`zk-service-remove.bat`

  ```shell
  @echo off
   
  REM #
  REM # 说明：卸载ZooKeeper的Windows服务
  REM # 注意：此脚本必须拷贝到ZooKeeper的根目录，否则运行报错
  REM # DATE：			2018-8-10 23:29:50
  REM # Author：			许亮
  REM #
   
  CD /d %~dp0
  %CD%\bin\prunsrv.exe //DS//ZooKeeper
  ```
  
* 停止服务脚本：`zk-stop.bat`

  ```shell
  @echo off
   
  REM #
  REM # 说明：以杀进程的方式停止ZooKeeper服务
  REM # 注意：此脚本必须拷贝到ZooKeeper的根目录，否则运行报错
  REM # DATE：	2018-8-10 16:51:16
  REM # Author：	许亮
  REM #
   
  setlocal
   
  CD /d %~dp0
   
  :: 以杀进程的方式停止ZooKeeper服务
  SET /p zkPID=<%CD%\log\zookeeper.pid
  taskkill /PID %zkPID% /T /F
   
  endlocal
  ```

  *以上脚本编码要设置为`ANSI` ,否则会有乱码*

* 以管理员身份运行 `zk-service-install.bat` 
* 在服务中找到ZooKeeper, 启动
* 查看ZooKeeper是否处于监听状态 :
```shell
  netstat -an | findstr 2181
```

**引用：** https://blog.csdn.net/xht555/article/details/81571389
记录一下以便查询









