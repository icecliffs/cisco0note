## 技术原理

ACLs的全称为**接入控制列表**（Access Control Lists），也称访问控制列表（Access Lists），俗称防火墙，在有的文档中还称包过滤。ACLs通过定义一些规则对网络设备接口上的数据包文进行控制；允许通过或丢弃，从而提高网络可管理型和安全性；
IP ACL分为两种：即**标准IP访问列表**和**扩展IP访问列表**，编号范围为（1～99、1300～1999、100～199、2000～2699）
标准IP访问控制列表可以根据数据包的源IP地址定义规则，进行数据包的过滤；
扩展IP访问列表可以根据数据包的原IP、目的IP、源端口、目的端口、协议来定义规则，进行数据包的过滤；
IP ACL基于接口进行规则的应用，分为：入栈应用和出栈应用；

## 项目一

### 拓扑图

![img](https://bfs.iloli.moe/img/2021/02/WeChat-Screenshot_20210226105034.png)

#### 说明

要求PC1可以访问PC2，PC0不能访问PC2，但PC2可以访问PC1

#### 地址表

![img](https://bfs.iloli.moe/img/2021/02/WeChat-Screenshot_20210226110017.png)

### 配置命令

#### Router0

```
# 配置IP地址
Router(config-if)#int fa0/0
Router(config-if)#no shutdown
Router(config-if)#ip address 192.168.2.100 255.255.255.0
Router(config-if)#int fa0/1
Router(config-if)#no shutdown 
Router(config-if)#ip address 192.168.1.1 255.255.255.0
Router(config-if)#int s0/0/0
Router(config-if)#no shutdown 
Router(config-if)#ip add 10.0.0.1 255.0.0.0
Router(config-if)#clock rate 72000
# 先配置RIP Version2 使全网互通
Router(config)#router rip 
Router(config-router)# v 2
Router(config-router)#network 10.0.0.0
Router(config-router)#no auto
Router(config-router)#network 192.168.1.0
Router(config-router)#network 192.168.2.0
# 配置ACL标准访问控制
Router(config)#access-list 1 deny 192.168.2.0 0.0.0.255
Router(config)#access-list 1 permit 192.168.1.0 0.0.0.255
```

[admonition] 说明：access-list [组名] deny/permit (deny为拒绝，permit为允许)，IP地址，子网掩码/反子网掩码 [/admonition]

```
# 接下来进入到PC2的接口，把刚才的ACL应用上
Router(config)#int fa0/1
Router(config-if)#ip access-group 1 out
# 访问控制列表出栈流量调用
```

#### Router1

```
Router(config)#int fa0/0
Router(config-if)#no shutdown 
Router(config-if)#ip add 192.168.3.100 255.255.255.0
Router(config-if)#int s0/0/0
Router(config-if)#no shutdown 
Router(config-if)#ip add 10.0.0.2 255.0.0.0 
Router(config-if)#clock rate 72000
Router(config)#router rip 
Router(config-router)#v 2
Router(config-router)#no auto
Router(config-router)#network 10.0.0.0
Router(config-router)#network 192.168.3.0
```

### 实验结果

![img](https://bfs.iloli.moe/img/2021/02/WeChat-Screenshot_20210226111024.png)

## 项目二

### 拓扑图

![img](https://bfs.iloli.moe/img/2021/02/WeChat-Screenshot_20210226112546.png)

#### 说明

假如你是一家公司的网络管理员，现在要求你对一家公司进行网络配置，要求，**外网**不能访问一家公司服务器上的WEB和DNS服务，而**内网**可以访问一家公司的WEB和DNS服务，且内网和外网都可以ping通一家公司

一家公司服务器信息如下

- 域名：www.yijiacompany.com
- IP：10.0.0.1

![img](https://bfs.iloli.moe/img/2021/02/WeChat-Screenshot_20210226112537.png)

#### 地址表

![img](https://bfs.iloli.moe/img/2021/02/WeChat-Image_20210226112925.png)

### 配置命令

#### Switch0

```
# 配置IP地址
Switch(config)#int fa0/2
Switch(config-if)#no switchport 
Switch(config-if)#ip address 192.168.0.100 255.255.255.0
Switch(config-if)#int fa0/1
Switch(config-if)#no sw
Switch(config-if)#ip add 192.168.1.100 255.255.255.0
Switch(config-if)#ip routing
Switch(config)#int g0/1
Switch(config-if)#no sw
Switch(config-if)#ip add 10.0.0.100 255.255.255.0
# 配置扩展访问控制列表
Switch(config)#ip access-list extended denyservice
Switch(config-ext-nacl)#deny tcp 192.168.0.0 0.0.0.255 10.0.0.0 0.0.0.255 eq www
Switch(config-ext-nacl)#deny tcp 192.168.0.0 0.0.0.255 10.0.0.0 0.0.0.255 eq domain
Switch(config-ext-nacl)#permit ip any any # 允许其他服务
Switch(config-ext-nacl)#exit
# 在目标接口上添加规则
Switch(config)#int fa0/2
Switch(config-if)#ip access-group denyservice in
```

[admonition]

说明：ip access-list extended [名字]

说明：[deny/permit] [各种协议] 宣告IP网段 反掩码 目标IP网段 反掩码 [大于/等于/小于等等] （eq代表equal即，等于） [服务名或端口号]

[/admonition]

### 实验结果

![img](https://bfs.iloli.moe/img/2021/02/WeChat-Screenshot_20210226140232.png)

## 项目三

### 拓扑图

![img](https://bfs.iloli.moe/img/2021/02/WeChat-Screenshot_20210226153521.png)

#### 说明

假如是你的员工，现在假如发现了黑客正在攻击一家公司的内网，为了提高一家公司网络的安全性，现在要求你对用户的IP和MAC进行网络访问控制

说明：PC0不能访问Server0的任何服务，PC1可以

#### 地址表

 

![img](https://bfs.iloli.moe/img/2021/02/WeChat-Image_20210226154408.png)

### 配置命令

#### Switch0

```
# 配置IP地址
Switch(config)#int fa0/1
Switch(config-if)#no sw
Switch(config-if)#ip add 192.168.1.100 255.255.255.0
Switch(config-if)#int fa0/3
Switch(config-if)#no sw
Switch(config-if)#ip add 192.168.2.1 255.255.255.0
Switch(config-if)#int fa0/2
Switch(config-if)#no shutdown 
Switch(config-if)#ip add 172.16.0.1 255.255.255.0
# 配置访问控制列表
Switch(config)#ip access-list extended test1
Switch(config-ext-nacl)#deny ip host 192.168.1.1 host 1.1.1.1
Switch(config-ext-nacl)#permit ip any any
Switch(config-ext-nacl)#int fa0/1
Switch(config-if)#ip access-group test1 in
Switch(config-if)#ip routing
Switch(config-if)#ip address 192.168.2.100 255.255.255.0
# 使全网互通哈
Switch(config)#ip route 1.1.1.0 255.255.255.0 fa0/2
```

#### Router0

```
# 配置IP
Router(config)#int fa0/0
Router(config-if)#no shutdown 
Router(config-if)#ip add 172.16.0.2 255.255.255.0
Router(config-if)#int fa0/1
Router(config-if)#no shutdown 
Router(config-if)#ip add 1.1.1.100 255.255.255.0
Router(config-if)#exit
# 配置静态IP是全网互通
Router(config)#ip route 192.168.1.0 255.255.255.0 fa0/0
Router(config)#ip route 192.168.2.0 255.255.255.0 fa0/0
```

## 案例加强

### 拓扑图

![img](https://bfs.iloli.moe/img/2021/02/WeChat-Screenshot_20210226161339.png)

#### 说明

子公司A只可以访问区域1的Server0上的DNS和FTP服务

子公司B只可以访问区域1的Server1上的WEB和DHCP服务

## 总结

access-list 用于配置访问控制列表
具体使用方法自己敲问号（❓）
ip access-group 1 [out/in] 应用访问控制列表

 

 

 

 

 

 

 