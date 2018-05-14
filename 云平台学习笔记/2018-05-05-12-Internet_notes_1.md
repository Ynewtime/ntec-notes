---
layout: post
share: true
title: 华为交换机/路由器-链路聚合配置方案
description:  基本知识
date: 2018-05-05
---

一、 基本知识 

1、概述 

链路聚合（Link Aggregation）是将—组物理接口捆绑在一起作为一个逻辑接口来增加带宽的一种方法。随着网络规模不断扩大，用户对骨干链路的带宽和可靠性提出越来越高的要求。在传统技术中，常用更换高速率的接口板或更换支持高速率接口板的设备的方式来增加带宽，但这种方案需要付出高额的费用，而且不够灵活。采用链路聚合技术可以在不进行硬件升级的条件下，通过将多个物理接口捆绑为一个逻辑接口，实现增加链路带宽的目的。链路聚合的备份机制能有效提高可靠性，同时，还可以实现流量在不同物理链路上的负载分担。 

如图所示，DeviceA与DeviceB之间通过三条以太网物理链路相连，将这三条链路捆绑在一起，就成为了一条逻辑链路Eth-trunk，这条逻辑链路的带宽等于原先三条以太网物理链路的带宽总和，从而达到了增加链路带宽的目的；同时，这三条以太网物理链路相互备份，有效地提高了链路的可靠性。 

![](https://wx1.sinaimg.cn/large/78905b2cgy1fqzunjk5n4j20at01n3yc.jpg)

2、手工负载分担模式链路聚合 

手工负载分担模式下，Eth-Trunk的建立、成员接口的加入完全由手工来配置。 

该模式下所有活动链路都参与数据的转发，平均分担流量，因此称为负载分担模式。如果某条活动链路故障，链路聚合组自动在剩余的活动链路中平均分担流量。

3、LACP模式链路聚合 

LACP模式是一种利用LACP协议进行聚合参数协商、确定活动接口和非活动接口的链路聚合方式。该模式下，需手工创建Eth-Trunk，手工加入Eth-Trunk成员接口，由LACP协议协商确定活动接口和非活动接口。LACP模式也称为M∶N模式。这种方式同时可以实现链路负载分担和链路冗余备份的双重功能。在链路聚合组中M条链路处于活动状态，这些链路负责转发数据并进行负载分担，另外N条链路处于非活动状态作为备份链路，不转发数据。当M条链路中有链路出现故障时，系统会从N条备份链路中选择优先级最高的接替出现故障的链路，并开始转发数据。 

![](https://ws1.sinaimg.cn/large/78905b2cgy1fqzup6hm99j20an03swed.jpg)

区别： 

LACP模式与手工负载分担模式的主要区别为：LACP模式有备份链路，而手工负载分担模式所有成员接口均处于转发状态，分担负载流量。

4、注意事项

a、每个Eth-Trunk接口下最多可以包含8个成员接口。  
b、成员接口不能配置任何业务和静态MAC地址。  
c、成员接口加入Eth-Trunk时，必须为缺省的Hybrid类型接口。  
d、Eth-Trunk接口不能嵌套，即Eth-Trunk接口的成员接口不能是Eth-Trunk接口。  
e、一个以太网接口只能加入到一个Eth-Trunk接口，如果需要加入其它Eth-Trunk接口，必须先退出原来的Eth-Trunk接口。  
f、一个Eth-Trunk接口中的成员接口必须是同一类型。  
g、如果本地设备使用了Eth-Trunk，与成员接口直连的对端接口也必须捆绑为Eth-Trunk接口，两端才能正常通信。、如果本地设备使用了Eth-Trunk，与成员接口直连的对端接口也必须捆绑为Eth-Trunk接口，两端才能正常通信。  
h、当成员接口加入Eth-Trunk后，学习MAC地址时是按照Eth-Trunk来学习的，而不是按照成员接口来学习。  
i、Eth-Trunk链路两端相连的物理接口的数量、速率、双工方式、jumbo、流控配置必须一致。 

二、配置及拓扑 

1、要求： 
 - 三台交换机和三台pc 
 - 交换机两两之间通过两根线缆连接 
 - pc之间通过vlan20通信 
 - pc1:192.168.8.1/24 
 - pc2:192.168.8.2/24 
 - pc3:192.168.8.3/24 

2、目标： 

通过高可用和负载均衡实现全网互通 

3、拓扑 

![](https://ws1.sinaimg.cn/large/78905b2cgy1fqzus97jbxj20te0hbabi.jpg)

三、配置 

pc机配置 

![](https://ws1.sinaimg.cn/large/78905b2cgy1fqzuss0u1oj20lp0etaan.jpg)

![](https://ws1.sinaimg.cn/large/78905b2cgy1fqzuuvqbfwj20lp0etdgf.jpg)

![](https://ws1.sinaimg.cn/large/78905b2cgy1fqzuv4ng2nj20lp0etjrz.jpg)

此时不做任何配置，三台pc互通 

![](https://ws1.sinaimg.cn/large/78905b2cgy1fqzuvb30k1j20lp0et3zg.jpg)

Stp协议默认是开启的 

![](https://ws1.sinaimg.cn/large/78905b2cgy1fqzuvk93jlj21100ov0va.jpg)

1、交换机一

```bash
//进入配置模式
system-view
//创建vlan
vlan 20
quit
//进入接口1，并配置模式为acess vlan为20
interface GigabitEthernet 0/0/1
port link-type access
port default vlan 20

//创建sw1和sw2之间的eth-trunk 12
interface Eth-Trunk 12
quit
//将接口2加入到eth-trunk中
interface GigabitEthernet 0/0/2
eth-trunk 12
quit
//将接口3加入到eth-trunk中
interface GigabitEthernet 0/0/3
eth-trunk 12
quit

//修改eth-trunk 12为trunk模式，vlan模式为20
interface Eth-Trunk 12
port link-type trunk
port trunk allow-pass vlan 20
quit
//创建sw1和sw3之间的eth-trunk 13
interface Eth-Trunk 13
port link-type trunk
port trunk allow-pass vlan 20
//将接口4加入到eth-trunk中
interface GigabitEthernet 0/0/4
eth-trunk 13
quit
//将接口5加入到eth-trunk中
interface GigabitEthernet 0/0/5
eth-trunk 13
quit
//修改eth-trunk 13为trunk模式，vlan模式为20
interface Eth-Trunk 13
port link-type trunk
port trunk allow-pass vlan 20
quit
```

2、交换机二

```bash
//进入配置模式
system-view
//创建vlan
vlan 20
quit
//进入接口1，并配置模式为acess vlan为20
interface GigabitEthernet 0/0/1
port link-type access
port default vlan 20

//创建sw1和sw2之间的eth-trunk 12
interface Eth-Trunk 12
quit
//将接口2加入到eth-trunk中
interface GigabitEthernet 0/0/2
eth-trunk 12
quit
//将接口3加入到eth-trunk中
interface GigabitEthernet 0/0/3
eth-trunk 12
quit

//修改eth-trunk 12为trunk模式，vlan模式为20
interface Eth-Trunk 12
port link-type trunk
port trunk allow-pass vlan 20
quit

//创建sw2和sw3之间的eth-trunk 23
interface Eth-Trunk 12
quit
//将接口2加入到eth-trunk中
interface GigabitEthernet 0/0/23
eth-trunk 23
quit
//将接口3加入到eth-trunk中
interface GigabitEthernet 0/0/24
eth-trunk 23
quit

//修改eth-trunk 23为trunk模式，vlan模式为20
interface Eth-Trunk 23
port link-type trunk
port trunk allow-pass vlan 20
quit
```

3、交换机三

```bash
system-view
//创建vlan
vlan 20
quit
//进入接口1，并配置模式为acess vlan为20
interface GigabitEthernet 0/0/1
port link-type access
port
port default vlan 20

//创建sw1和sw2之间的eth-trunk 13
interface Eth-Trunk 13
quit
//将接口2加入到eth-trunk中
interface GigabitEthernet 0/0/4
eth-trunk 13
quit
//将接口3加入到eth-trunk中
interface GigabitEthernet 0/0/5
eth-trunk 13
quit

//修改eth-trunk 13为trunk模式，vlan模式为20
interface Eth-Trunk 13
port link-type trunk
port trunk allow-pass vlan 20
quit
//创建sw2和sw3之间的eth-trunk 23
interface Eth-Trunk 23
quit
//将接口2加入到eth-trunk中
interface GigabitEthernet 0/0/23
eth-trunk 23
quit
//将接口3加入到eth-trunk中
interface GigabitEthernet 0/0/24
eth-trunk 23
quit

//修改eth-trunk 23为trunk模式，vlan模式为20
interface Eth-Trunk 23
port link-type trunk
port trunk allow-pass vlan 20
quit
```

三、查看配置 

1、交换机一 

接口状态 

![](https://ws1.sinaimg.cn/large/78905b2cgy1fqzv3aco0yj20iw0a0dfy.jpg)

Vlan状态 

![](https://ws1.sinaimg.cn/large/78905b2cgy1fqzv3lpbftj20iz0dht8y.jpg)

2、交换机二 

查看接口状态 

![](https://ws1.sinaimg.cn/large/78905b2cgy1fqzv3zau8kj20it0akjrl.jpg)

Vlan状态 

![](https://ws1.sinaimg.cn/large/78905b2cgy1fqzv447l8aj20il0di74i.jpg)

3、交换机三

查看接口状态 

![](https://ws1.sinaimg.cn/large/78905b2cgy1fqzv4lu13dj20iy0aeaa9.jpg)

Vlan状态 

![](https://ws1.sinaimg.cn/large/78905b2cgy1fqzv4qqqibj20ez02qgli.jpg)

四、测试： 

1、配置链路聚合后，sw1和sw2直接的链路由于stp协议，为避免网络风暴端口被discarding，全网互通。 
sw1 

![](https://ws1.sinaimg.cn/large/78905b2cgy1fqzv5djy0aj20g202xmx2.jpg)

sw2

![](https://ws1.sinaimg.cn/large/78905b2cly1fqzv5imarbj20eq03oglj.jpg)

sw3

![](https://ws1.sinaimg.cn/large/78905b2cly1fqzv5ydxv7j20ez02qgli.jpg)

![](https://ws1.sinaimg.cn/large/78905b2cly1fqzv64ultvj20tc0h13zz.jpg)

2、down掉sw3和sw2之间一根线缆，全网互通 

![](https://ws1.sinaimg.cn/large/78905b2cly1fqzv6ixpjfj20u90hbwfz.jpg)

3、down掉sw3和sw2之间两个根线缆，全网互通 

![](https://ws1.sinaimg.cn/large/78905b2cly1fqzv6pvgj9j20t90hegn1.jpg)

4、down掉sw3和sw2之间两个根线缆，sw1和sw2之间一根线缆，全网互通 

![](https://ws1.sinaimg.cn/large/78905b2cgy1fqzv6tkre1j20t90go75m.jpg)

5、down掉sw3和sw2之间两个根线缆，sw1和sw2之间一根线缆，sw1和sw3之间一根线缆，全网互通。 

![](https://ws1.sinaimg.cn/large/78905b2cgy1fqzv75vqtgj20te0h2jso.jpg)