---
layout: post
share: true
title: VLAN路由 - 工作原理
description: VLAN的缺点/VLAN间通信
date: 2018-05-03
---

### VLAN的缺点

VLAN隔离了二层广播域，也就严格地隔离了各个VLAN之间的任何流量，分属于不同VLAN的用户不能相互通信。

VLAN路由也就是为了实现不同VLAN之间的数据通信而产生的。

### VLAN间通信

一、借助路由器的三层路由功能实现不同VLAN之间的通信。

📌 适用于VLAN数较少的情况

```
VLAN100
PC1----
      |
      |Port1           Ethernet0
      --------SW1--------R1（相当于一个网关）
      |Port2           Ethernet1
      |
PC2----
VLAN200
```

二、使用VLAN Trunking（独臂路由的方式）

二层交换机和路由器之间项链的接口配置VLAN Trunking，使多个VLAN共享同一条物理链路连接到路由器

📌 适用于VLAN数较多的情况

```
VLAN100
PC1----
      |
      |        -->Trunking-->
      --------SW1--------R1
      |          Ethernet0.100
      |          Ethernet0.200（子接口）
PC2----
VLAN200
```

三、交换和路由的集成

二层交换机和路由器在功能上的集成构成了三层交换机  

三层交换机在功能上实现了VLAN的划分、VLAN内部的二层交换和VLAN间路由的功能

```
PC1-----      ----PC2
       |      |
       |      |
       --------
           |
           |
          SW1（三层交换机）
           |
           |
          PC3
```

📌 实现方式：通过配置VLAN Interface（即Vlanif）的方式来实现。