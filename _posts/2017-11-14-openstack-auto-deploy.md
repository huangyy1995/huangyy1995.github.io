---
layout: post
title: "OpenStack的自动部署实现"
date: 2017-11-14T22:45:21+08:00
categories:
  - programming
  - project
tags:
  - random
---

本文介绍的是我的一个课程项目**OpenStack的自动部署实现** ，附[英文版详细报告详细报告](<https://github.com/vulture29/OpenStack-auto-deploy/raw/master/Document.pdf>)以及[代码](<https://github.com/vulture29/OpenStack-auto-deploy>)。

# 项目介绍

OpenStack是一个免费开源的云计算平台项目，它提供的主要是IaaS(Infrastructure as a service)的解决方案，也就是提供plain虚拟机资源。基于OpenStack我们可以构造出大规模的云计算数据中心，它在工业界被应用的十分广泛。然而他的安装和调试比较复杂琐碎，对他的构造和维护需要比较大的代价。对于一些小型工作室，甚至个人而言，他们可能更关心的是如何快速构造一个可用的OpenStack，这个项目的目的就是基于PackStack编写一套可以快速构造一个可调试的OpenStack系统。

功能要求：

  * OpenStack的快速安装
  * 用户可以自定义配置信息
  * 自动卸载
  * 自定义OpenStack内的网络
  * 可以通过Web接口控制
  * 身份验证功能
  * 用户可管理镜像资源
  * 可扩展性（支持添加新硬件）



# OpenStack相关介绍

## OpenStack的几大组件

![OpenStack_service](/content/images/2017/11/openstack_service.png)

  * Nova(Compute)
    * 核心部分，整个IaaS服务的主要部分，它负责提供计算服务，也就是提供负责生产虚拟机以及一部分他们之间的基本通信
  * Neutron(Network)
    * 负责SDN，提供Network as a Service。用户可以通过Neutron API自定义网络，控制他们的流量，连接不同网络内的虚拟机，它还有插件一些高级功能比如load balancing, firewalls, VPN
  * Horizon(Dashboard)
    * 前端web界面。可以通过Web界面对OpenStack进行控制。当然提供的功能没有Terminal那么丰富。
  * Cinder(Block Storage)
    * 提供Block存储，是必须的组件之一。负责Block内的所有存储，包括虚拟机所需的block空间。
  * Swift(Object Storage)
    * 是一个可选择的组件。负责Object Storage
  * Keystone(Identity)
    * 负责身份验证功能
  * Glance(Image)
    * 负责管理虚拟机的镜像



## PackStack

PackStack是有Redhat发行的一个OpenStack安装工具，我们将使用PackStack对OpenStack进行安装。

# 实现

## 系统及环境依赖

  * CentOS 7
  * 节点最低配置
    * 网络节点(1 CPU, 1GB RAM, 5 GB of storage)
    * 计算节点(2 CPU, 2GB RAM, 10 GB of storage)
    * 控制节点(1 CPU, 2GB RAM, 5 GB of storage)



## 基本步骤

  * 读入用户输入并产生配置文件
  * 基于用户配置，检查节点间网络连通性
  * 清理可能影响到安装过程的残留文件
  * 检查系统及环境是否符合OpenStack安装条件
  * 安装开始
  * 结果
    * 若安装成功，显示成功信息并提供web接口的URL
    * 若安装失败，提示错误信息


    
    
    “Do you want to proceed with default configuration? [y/n]”  
    → If user input is “no” then first check with the customized config file is present or not.  
      → If customize config file is present then prompt the user.  
        “Do you really want to proceed with config file (PATH) ?  
        → If yes then  
          proceed the installation with the customized file.  
        → else  
          prompt the user for input (configuration)  
      → else  
        script should prompt for user input for configuration  
        For ex:  
          ● Please enter the controller node IP address.  
          ● Do you want to enable cinder service? [y/n].  
    → else  
      install the OpenStack with default configuration.  
      
  
---  
  
## 流程图

![flow_chart](/content/images/2017/11/flow_chart.jpg)

# 案例

## 环境

本工具在NC State网络实验室测试安装，实验室的硬件包括：

  * Dell Power-Edge T610服务器 2台(四核处理器, 24GB内存, 1TB硬盘空间)
  * Dell Power-Edge T310服务器 2台(双核处理器, 4GB内存, 300GB硬盘空间)
  * Cisco SG 300 20-port Gigabit Managed Switch 交换机
  * TP-LINK TL-WN722N Wireless USB adapter 无线网卡
  * KVM
  * CentOS 7



## 架构设计及实现

### 三节点架构

我们参考了OpenStack官方文档，最终决定采用三节点的基本架构(Compute-Network-Controller)

![OpenStack_arch](/content/images/2017/11/openstack_arch.png)

此设计的有点包括:

  * 由于每个节点提供不同的服务，三节点架构可以提供更好的可扩展性，可靠性和可用性
  * 把节点分离出来可以使得整个系统的性能提升
  * 把Network节点分离出来可以让网络的隔离性更好，也更加安全



### 控制节点(Controller Node)

控制节点负责处理OpenStack架构中的大部分服务。Keystone，Glance，Nova API，Neutron API都安装在控制节点上。控制器点上还安装了数据库和消息传递等服务。如果未安装和配置对象存储，则Glance服务将使用本地存储来存储用于启动虚拟机的映像。

### 网络节点(Network Node)

此节点负责处理Internet，OpenStack组件和物理计算节点上的vm之间的所有传入和传出流量。由于网络内容复杂，所以需要为环境提供一定的隔离。该节点还安装了neutron DHCP agent服务，它将为vm动态分配公共IP。

### 计算节点(Compute Node)

该节点负责处理按需分配虚拟机的各种配置。若不使用提供块处理的Cinder服务，可以使用DAS(Direct Attached Storage)以使得结构简单。

### 网络架构

以下是三节点架构下网络的基本架构示意图:

![OpenStack_network](/content/images/2017/11/openstack_network.png)

#### 管理网络(Management Network)

所有的节点都会通过一个共同的管理网络相连。这个网络将会负责传输关于启动或者停止OpenStack服务的请求，创建新虚拟机，身份验证，更改现有硬件配置等等的消息。在实验室网络下，控制节点，网络节点和计算节点的IP分别是192.168.0.42/24, 192.168.0.22/24 and 192.168.0.32/24。这个也是脚本的缺省值。

#### 封装网络(GRE Tunnel / VM Data Network)

GRE(Generic Routing Encapsulation)管道即通用路由封装管道，可实现点对点的链路。我们将网络节点和计算节点通过管道相连。若有多个计算节点，每一个都要与网络节点建立一条独立的管道。这条管道的网络流量是私有的。它负责承载来自外部网络与虚拟机之间相互通信的流量，由于外部网络只直接连接网络节点，网络节点需要将流量进行转发，而这个过程中的数据应该有较好的独立性，不应该被其他流量所干扰，所以我们设立了一条单独的GRE管道。

#### 外部网络(External Network)

与网络节点相连，主要用于将整个系统连接到互联网。实验室通过在网络节点上插入无线网卡访问外部网络。NAT和网络配置是在每个节点需要访问互联网的物理服务器上完成的。

#### 计算节点内部网络

![compute_network](/content/images/2017/11/compute_network.png)

计算节点内运行着一个OpenVSwitch。OpenStack安装时会自动设置一个默认的Linux网桥(virbr0).这个网桥会与一个网络接口相连(比如eth0, eth1)。当Nova-compute模块启动一个新vm时该vm会创建一个与该网桥相连的端口，这个端口是计算节点和vm网络通信的媒介。

#### 网络节点内部网络

![network_network](/content/images/2017/11/network_network.png)

网络节点内有一个虚拟router。当一个子网中的虚拟机希望与另一个运行在不同子网中的虚拟机通信时，该虚拟设备将提供所需的路由规则。

#### 创建vm请求示例

我们以当客户发出一个创建VM的请求为例,说明OpenStack中消息的传递过程:

![flow](/content/images/2017/11/flow.gif)

### 实现

#### 单服务器实现

一个物理服务器用于托管整个云栈，其中三个主节点 - 网络，计算和控制节点被安装或作为单独的虚拟机运行。 Hypervisor需要在物理服务器上运行虚拟机。 OpenStack支持XEN，KVM等许多管理程序。本项目中我们使用KVM做为管理程序来创建和运行这些VM。

![network_network](/content/images/2017/11/network_network.png)

优点:

  * 由于所有”节点”都运行在一个物理服务器上，他们之间的通信会比多服务器实现快
  * 由于整个系统在一个服务器上，我们可以方便的存储系统的某一状态作为镜像，还可以方便的回滚
  * 由于所有的节点都是虚拟机，如果某一节点出现比较严重的错误，我们可以直接重新创建一个新的vm，这样比实体服务器中必须要重装系统方便很多



缺点:

  * 最大的缺点是性能问题，由于一台服务器的性能有限，整个系统的性能也会有瓶颈。而且，由于计算节点还会产生新的虚拟机，这样的二重结构虚拟机的速度也会比较慢
  * 单服务器架构的扩展性也受到限制，增加计算节点的数量可能会导致性能进一步下降
  * 由于所有的节点都在一台物理服务器上，不同节点之间的隔离性比多服务器差



#### 三服务器实现

在我们的多服务器实现中，三台物理计算机充当网络，计算和控制器节点。这三个服务器由交换机管理网络连接。这种实现的应用更加广泛。

![network_network](/content/images/2017/11/network_network.png)

优点:

  * 性能好
  * 可扩展性好
  * 更好的隔离性，更加安全



缺点:

  * 由于物理机的数量增加，安装和维护成本将会比单台服务器结构更高。



#### 扩展

我们可以扩展计算节点的数量，没增加一个计算节点，我们需要将其加入管理网络并建立一个与网络节点的GRE管道。

# 总结

通过本项目，我学到的最多的是整个系统的架构设计以及各种情况的优劣和trade-off。

安装过程基于的是强大的PackStack，实际上这个工具有比较多的前提条件，加入了这些前提条件后可以进一步完善安装过程。

当然，整个过程中也学习到了OpenStack的内部结构和基本实现原理.
