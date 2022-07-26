### 详解ip地址
#### 局域网的概念
交换机：用于连接内网的多台设备，组成局域网
路由器：用于连接本地局域网和互联网
家用路由器：三层路由器，是路由器和交换机的集合体

#### ip地址
ip 地址由32位的二进制数字组成，最大255.255.255.255

#### 子网掩码
ip地址和子网掩码互相配合使用，有ip的地方必须使用子网掩码

局域网通信规则：在同一个局域网中，所有ip必须在同一网段才可以相互通信，如`10.1.1.1` 和`20.0.0.1`无法相互通信

ip地址构成：网络位+主机位，网络位相同的ip地址，为同一网段

子网掩码：用来确定ip地址的网络位

如何确定网络位：与255对应的数字为网络位，与0对应的数字为主机位，如`255.0.0.0` `255.255.0.0` `255.255.255.0`


如：一个教室为一个网段，如果不同教室为同一个网段，广播量太大，需要有二层路由器的接入才可以完成广播，组成局域网的交换机无法完成

#### ip地址详解
国际标准ios定位地址分类：五大类，是以ip地址的第一位进行区分的

A类：1-126，默认子网掩码 `255.0.0.0`
B类：128-191，默认子网掩码 `255.255.0.0`
C类：192-223，默认子网掩码 `255.255.255.0 `
D类：224-239，组播地址，ABC类为单播地址，一个地址对应一个机器；组播地址对应一组机器，同一个数据发送方只需要向组播地址发送一次
E类：240-254，科研使用

注
1. 目前我们可以使用的只有 A B C 三类地址
2. A B C 三类地址的子网掩码可以修改

每个网段有三种地址
1. 网段名称，把该网段的主机位全部置0
2. 广播地址，该网段的主机位全为255
3. 剩下的为可用ip地址

案例1
ip`10.1.1.1`，子网掩码`255.255.255.0`，ip属于什么网段？所在网段有多少个可用的ip地址？该网段的广播地址是什么？
1. `10.1.1.1`属于`10.1.1.0`网段
2. `10.1.1.0`网段可用的ip地址范围：`10.1.1.1`~`10.1.1.254`
3. `10.1.1.0`网段的广播地址：`10.1.1.255`，代表该网段的所有人，交换机会将该ip地址收到的网络包向所有人发送
4. `255.255.255.255`全局广播地址，进行网络隔断后，`10.1.1.255`和`255.255.255.255`效果一致

案例2
ip`10.1.1.1`，子网掩码`255.255.0.0`，ip属于什么网段？所在网段有多少个可用的ip地址？该网段的广播地址是什么？
1. `10.1.1.1`属于`10.1.0.0`网段
2. `10.1.0.0`网段可用的ip地址范围：`10.1.0.1`~`10.1.255.254`
3. `10.1.1.0`网段的广播地址：`10.1.255.255`

`127.0.0.1` 回环地址，代表本机，如可用于检查网卡是否损坏 

#### 网关
网关：一个网络的出口，GateWay=GW，一般网关是在路由器上

路由器：可用于连接内外网的设备

路由器在内网局域网内有一个ip地址，习惯用于该网段的第一个或最后一个ip作为网关，路由器该接口也是局域网的一部分，称为内网的出口

pc向外发包
1. 判断目标ip地址与自己是否为同一网段
2. 如在同一网段，则直接发出去，不找网关
3. 不在同一网段，则直接发包给网关

#### DNS
DNS: Domain Name Service 域名服务

##### 测试网络连通性
```shell
ping ip地址或域名
ping -t ip          # 一直ping
ping -n 数字 ip地址  # 修改ping包数量
ping -l 数字 ip          # 数据报的长度
```

##### 手工解析域名
`nslookup www.baidu.com`

#### DHCP
##### 概念
主机ip自动分配协议，减少工作量，避免ip冲突、提高ip地址利用率

##### 相关概念
地址池/作用域：ip、子网掩码、网关、DNS、租期

##### DHCP原理
也称为DHCP租约过程
1. 发送DHCP Discovery广播请求：客户机不知道DHCP服务在哪，广播请求IP地址，包含客户机的MAC地址
2. 相应DHCP Offer广播包：提供ip地址，但无子网掩码、网关等参数
3. 客户机发送DHCP Request广播包：可能有收到了多个DHCP服务器的相应，确定ip后请求后续的信息
4. DHCP服务器发送DHCP ACK广播包：服务器确定了租期，更新租约配置表和地址池信息，并提供ip相关详细参数

DHCP攻击
1. 伪装mac地址，不断请求ip地址。防御：功能型交换机，动态mac地址绑定，在第一次请求时，记录ip和mac地址对应关系
2. 伪装DHCP服务器，分配错误的ip地址，导致在ip地址和局域网网段不同，请求无法发送出去。防御：功能性交换机，组织任何人发送offer包，除了指定的交换机

##### DHCP续约
当使用时间超过50%，客户机再次发送DHCP Request包，从此刻开始算，向后续约一个租期的时间，一般DHCP服务器会返回一个ACK包

公共场合，比如肯德基，DHCP租期短一点好，相应的DHCP服务器会较为繁忙

如果设备关机了，开机会发送DHCP Request包，DHCP发现租期没到，继续分配该ip地址

如果续约时DHCP服务器无响应，则继续在87.5%租期时，发送的DHCP Request包进行续约，如果仍无响应，清除网卡的ip相关信息，重新发送DHCP Discovery包申请ip地址

当获取不到ip地址时，网卡会给自己设置一个无效的ip地址 `169.254.x.x/16`，属于全球统一的无效地址，用于局域网临时通信

##### 部署DHCP服务器
1. 