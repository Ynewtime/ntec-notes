---
layout: post
share: true
title: VLAN技术原理和配置 - VLAN端口类型与配置
description: ACCESS接口VLAN属性/配置ACCESS接口属性/TRUNK接口VLAN属性/配置TRUNK接口属性/Hybrid端口VLAN属性/配置HYBRID属性/端口VLAN属性动态配置/配置GVRP
date: 2018-05-03
---

### ACCESS接口VLAN属性

```
[Quidway]display port vlan active GigabitEthernet 0/0/1
T=TAG U=UNTAG
---------------------------------------
Port     Link Type    PVID   VLAN List
---------------------------------------
GE0/0/1  access       2      U:2
           |          |       |
           |          |       ---->接口允许通过的VLAN与PVID相同
    Access接口，一般用于连接主机
                      |
                      |
                接口默认VLAN为2
            UNTAGGED帧添加VLAN 2后再转发
```

对于主机发向该端口的数据帧，端口要添加上对应的PVID

对于设备发向该端口的数据帧，端口要匹配PVID，相同时去标签转发给用户


### 配置ACCESS接口属性

```
//配置ACCESS接口类型
[SWA-Ethernet0/1]port link-type access

//创建VLAN
[SWA]vlan 3

//设置ACCESS接口PVID
[SWA-Ethernet0/1]port default vlan 3
```

### TRUNK接口VLAN属性

```
[Quidway]display port vlan active Ethernet 0/3
T=TAG U=UNTAG
---------------------------------------
Port     Link Type    PVID   VLAN List
---------------------------------------
E0/0/1   trunk        2      U:2
           |          |      T:5---->允许多个VLAN通过
           |          |       
    用于连接交换机等网络设备
                      |
                      |
            收到UNTAGGED帧后添加PVID 3
```

🔨 处于接受数据帧的情况时：
 - 对于UNTAGGED的数据帧，端口要添加上对应的PVID
 - 对于TAGGED的数据帧，端口要匹配VLAN List，确认是否转发

🔨 处于发送数据帧的情况是：
 - 首先根据目的地址打上标签；
 - 然后再根据VLAN List对将要发送的数据帧做相应的处理（与该端口的PVID相同时去除标签转发，确认且不同时允许转发）

### 配置TRUNK接口属性

```
//配置TRUNK接口类型
[SWA-Ethernet0/1]port link-type trunk

//创建VLAN
[SWA]vlan 3

//设置TRUNK接口PVID
[SWA-Ethernet0/1]port trunk pvid vlan 3

//配置TRUNK-LINK所允许通过的VLAN（Permitted VLAN）
[SWA-Ethernet0/1]port trunk pvid allow-pass vlan 5
```

### Hybrid端口VLAN属性

```
[Quidway]display port vlan active Ethernet 0/3
T=TAG U=UNTAG
---------------------------------------
Port     Link Type    PVID   VLAN List
---------------------------------------
E0/0/1   hybrid        2      U:1 4
                       |      T:5---->移除标签后转发
                       |       
                       |
                       |
            按照Trunk端口的方式转发
```

🔨 优点：
1. 既可用于用户与设备之间，也可用于设备与设备之间，不受场景限制
2. 自定义端口，实现起来比较灵活
3. 实际使用较多🙈

### 配置HYBRID属性

🔨 物理拓扑

```
         SWA
          |
          |---->Port-2/0/0
         SWB
          |
Port-1/0/1|Port-1/0/24
        ------
        |    |
        |    |
       PC1  PC2
```

🔨 命令行配置

```
//配置hybrid接口类型
[Quidway-Ethernet1/0/1]port link-type hybrid
//设置hybrid接口PVID
[Quidway-Ethernet1/0/1]port hybrid pvid vlan 2
//配置hybrid-LINK的VLAN List
[Quidway-Ethernet1/0/1]port hybrid pvid untagged vlan 2 99

[Quidway-Ethernet1/0/24]port link-type hybrid
[Quidway-Ethernet1/0/24]port hybrid pvid vlan 3
[Quidway-Ethernet1/0/24]port hybrid pvid untagged vlan 3 99

[Quidway-Ethernet2/0/0]port link-type hybrid
[Quidway-Ethernet2/0/0]port hybrid pvid vlan 99
[Quidway-Ethernet2/0/0]port hybrid pvid untagged vlan 2 3 99
```

### 端口VLAN属性动态配置

🔨 过渡交换机添加VLAN信息
 - 手动配置（只适合小型网络）
 - 自动配置（GVRP)

### 配置GVRP

```
[Switch]gvrp
[Switch]intterface e0/1
[Switch-Ethernet0/1]port link-type trunk
[Switch-Ethernet0/1]port trunk allow-pass vlan all
[Switch-Ethernet0/1]gvrp
```

🔨 配置GVRP的接口会根据收到的数据创建对应的VLAN信息

---

📌 GVRP是通用VLAN注册协议（General VLAN Register Protocol），它是GARP（General Address Resolution Protocol）的一个应用。