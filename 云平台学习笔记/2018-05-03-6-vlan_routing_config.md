---
layout: post
share: true
title: VLAN路由 - 简单配置
description: 单臂路由配置/交换机配置/路由器配置/检测连通性
date: 2018-05-03
---

### 单臂路由配置

```
IP:192.168.10.10
GW:192.168.10.1
   |
   |
VLAN100
PC1----
      |
      |Port1     Port24   -->Trunking-->
      --------SW1----------------------------R1
      |Port2                       Ethernet0/1.1
      |                            Ethernet0/1.2
PC2----
VLAN200
   |
   |
IP:192.168.20.20
GW:192.168.20.1
```

### 单臂路由配置 - 交换机配置

```
//配置下行VLAN信息
[SW1]vlan 100
[SW1-vlan100]port ethernet 0/1
[SW1]vlan 200
[SW1-vlan200]port ethernet 0/2
//配置上行Trunk信息
[SW1]interface ethernet 0/24
[SW1-Ethernet0/24]port link-type trunk
[SW1-Ethernet0/24]port trunk allow-pass vlan all
```

### 单臂路由配置 - 路由器配置

```
//配置第一个子接口
[RT1]interface ethernet 0/1.1
[RT1-Ethernet0/1.1]control-vid 100 dot1q-termination  //对子接口控制、封装
[RT1-Ethernet0/1.1]dot1q-termination vid 100  //vlan的封装号为100
[RT1-Ethernet0/1.1]arp broadcast enable  //开启ARP广播消息传递
[RT1-Ethernet0/1.1]ip address 192.168.10.1 255.255.255.0  //将子接口的IP地址配置为PC1的网关
//配置第二个子接口
[RT1]interface ethernet 0/1.2
[RT1-Ethernet0/1.2]control-vid 200 dot1q-termination  //对子接口控制、封装
[RT1-Ethernet0/1.2]dot1q-termination vid 200  //vlan的封装号为200
[RT1-Ethernet0/1.2]arp broadcast enable  //开启ARP广播消息传递
[RT1-Ethernet0/1.2]ip address 192.168.20.1 255.255.255.0  //将子接口的IP地址配置为PC2的网关
```

### 单臂路由配置 - 检测连通性

VLAN100里的主机192.168.10.10使用ping命令测试与VLAN200里的主机192.168.20.20的连通性

---

📌 不同VLAN之间流量不能互通，需要使用VLAN路由实现互通。
📌 在VRP平台上，命令`interface vlaninterface vlanid`的作用是 生成或进入VLAN虚接口视图。