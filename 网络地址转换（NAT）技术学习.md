之前写了一大堆后来发现写的都是错的，lol，笑死我了，还好现在学完了来总结一下

------

起初这个（NAT）是为了推迟可用IP地址空间耗尽的时间，通俗易懂的讲NAT就是利用网关，将不能访问外部网络的IP地址转换为可访问外部的IP地址，从而实现内部网络到外部网络的访问，通常（NAT）用于边界路由器，尽管这样NAT也有不少的优缺点

- 优点
  - 节省合法的注册地址
  - 在地址重叠时提供解决方案
  - 连接到特网的灵活性
  - 在网络发生变化时避免重新编址
- 缺点
  - 地址转换将增加交换延迟
  - 导致无法进行端到端IP跟踪
  - 导致有些应用程序无法正常运行

[alert]本篇文章基于IPv4编写；但现今IPv4地址池已经耗尽，等到将来IPv6出来会在文末补充[/alert]

### NAT地址转换类型

静态NAT：这种 NAT 能够在本地地址和全局地址之间进行一对一的映射，**网络中的每台主机都必须要有一个公网IP地址**

动态NAT：它能够将未注册的 IP 地址映射到注册 IP 地址池中的一个地址，**必须有足够的公网 IP 地址**，让连接到因特网的主机都能够同时发送和接收分组

NAT重载：NAT 重载也是动态 NAT ，它利用源端口将多个非注册 IP 地址映射到一个注册 IP 地址（多对一），它也被称为端口地址特换即（PAT），通过使用 PAT（NAT 重载），只使用一个全局地址，就可将数千名用户连接到因特网。

### NAT地址常用术语

- 内部本地地址 - 转换前的内部源地址
- 内部全局地址 - 转换后的源地址
- 外部本地地址 - 转换后的目标地址
- 外部全局地址 - 转换前的外目标地址

#### 基本NAT转换

为了向互联网发送分组，主机A将数据包发给配置了 NAT 的边界路由器。该路由器发现分组的源地址为内部本地地址，且是前往外部网络的，因此对源地址进行转换，并将这种转换记录保存到 NAT 表中。然后，该分组被转发到外部接口，它包含转换后的源地址，收到外部主机返回的分组后，NAT 路由器根据 NAT 表将分组包含的内部全局地址转换为内部本地 IP 地址。

- 主机A：
  - 内部本地IP地址：10.1.1.1
  - 内部全局IP地址：11.45.14.19
- 主机B：
  - 内部本地IP地址：10.1.1.2
  - 内部全局IP地址：11.45.14.20
- 主机C：
  - 内部本地IP地址：10.1.1.3
  - 内部全局IP地址：11.45.14.21

#### NAT重载转换

还是刚才的IP，只不过现在有了一些变化，使用重载时，转换后的所有内部主机都使用同一个 IP 地址，除内部本地 IP 地址和外部全局 IP 地址外，它还包含端口号。这些端口号让路由器能够确定应该将返回的数据流转发给哪台主机。

- 主机A：
  - 内部本地IP地址：10.1.1.1
  - 内部全局IP地址：11.45.14.19
- 主机B：
  - 内部本地IP地址：10.1.1.2
  - 内部全局IP地址：11.45.14.20
- 主机C：
  - 内部本地IP地址：10.1.1.3
  - 内部全局IP地址：11.45.14.21

### 静态NAT配置

主要配置命令和配置方法如下

R-SIDE(config)#**ip nat** [inside/outside] source **static** [内部本地地址] [内部全局地址]

- ip nat inside source 内部接口指定为源，即转换的起点
- ip nat outside source 从而将外部接口指定为源，即转换的起点

R-SIDE(config-if)#ip nat inside # 设置内部接口

R-SIDE(config-if)#ip nat outside # 设置外部接口

[alert]R-SIDE为Router0[/alert]

```
R-SIDE(config-if)#int fa0/0
R-SIDE(config-if)#ip nat inside 
R-SIDE(config-if)#int s0/0/0
R-SIDE(config-if)#ip nat outside 
R-SIDE(config)#ip nat inside source static 10.0.0.1 152.23.36.3
```

这时候你会发现Router2可以ping通152.23.36.3这个IP地址，同时PC0也可以ping通152.23.36.2

### 动态NAT配置

配置动态 NAT 需要有一个地址池，用于给内部用户提供公有 IP 地址，动态 NAT 不使用端口号，因为所有试图访问外部网络的用户，都需要有一个公有 IP 地址

命令 ip nat inside source list 1 pool [pool name] 与 access-list 相匹配IP地址转换为 NAT 地址池中的一个可用地址。

ip nat pool lopu 152.23.36.3 152.23.36.254 netmask 255.255.255.0 创建一个地址池，用于将全局地址分配给主机。

```
POPU(config-if)#int s0/0/0
POPU(config-if)#ip nat outside 
POPU(config-if)#int fa0/0
POPU(config-if)#ip nat inside 
POPU(config)#ip nat pool lopu 152.23.36.3 152.23.36.254 netmask 255.255.255.0
POPU(config)#ip nat inside source list 1 pool lopu
POPU(config)#access-list 1 permit 10.0.0.0 0.0.0.255
```

 

 

 

 

 

 

 

 

 