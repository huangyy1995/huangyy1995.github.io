---
layout: post
title: "收集MapReduce和Spark的Metrics"
date: 2018-03-13T11:38:46+08:00
tags:
  - distributed system
---

随着大数据的普及，MapReduce和Spark也被运用的越来越广泛，有些时候我们为了更好的了解我们的历史执行记录，甚至分析我们的历史，我们需要获取平台的metrics.本篇内容针对MapReduce v2，也就是Yarn，关于新旧版本MapReduce区别以及Yarn介绍可参考<https://www.ibm.com/developerworks/cn/opensource/os-cn-hadoop-yarn/>

# 基本概念

## MapReduce

可参考:  
<http://blog.csdn.net/v_july_v/article/details/6637014>

## Spark

Spark的最大特点就是引入了RDD，可参考  
<http://blog.csdn.net/u011204847/article/details/51010205>

# 数据收集

## MapReduce

MapReduce的架构图如下：  
![yarn_arch](/content/images/2018/03/yarn_arch.png)  
资源管理器(Resource Manager)全局管理所有应用程序计算资源的分配，每一个应用的 ApplicationMaster 负责相应的调度和协调，NodeManager 是每一台机器框架的代理，是执行应用程序的容器，监控应用程序的资源使用情况 (CPU，内存，硬盘，网络 ) 并且向调度器汇报。当一个任务完成后，该任务的具体执行的log，例如average map time, average shuffle time传至history server保存。

我们获取MapReduce系统的metrics本质上就是获取这几个服务的metrics。

  * resource manager的默认端口号是8080，我们可以通过访问 ip+端口号 通过UI查看系统的metrics  
![sample_resource_manager](/content/images/2018/03/sample_resource_manager.png)  
我们还可以通过restful的方式获取数据  
![rest_resource_manager](/content/images/2018/03/rest_resource_manager.png)
  * node manager的默认端口号是8042，我们可以通过与resource manager类似的方式获取数据
  * history server的默认端口号是19888



具体api链接可参考[yarn官方文档](<https://hadoop.apache.org/docs/r2.4.1/hadoop-yarn/hadoop-yarn-site/WebServicesIntro.html>)

## Spark

与MapReduce类似，Spark也提供了Web UI和RESTful两种方式来获取数据，值得一提的是，已运行完的application将统一存在history server里，而每一个正在运行的application占用一个端口号显示其context

  * history server的默认端口号是18080
    * Web UI访问  
![sample_spark](/content/images/2018/03/sample_spark.png)
    * RESTful访问  
![rest_spark](/content/images/2018/03/rest_spark.png)
  * context的默认起始端口号为4000，即第一个运行的任务占用4000端口，第二个占用4001，以此类推



我们还可以访问每个spark application更细节的metrics，例如job，task，stage信息。具体api链接可参考[Spark官方文档](<https://spark.apache.org/docs/latest/monitoring.html>)
