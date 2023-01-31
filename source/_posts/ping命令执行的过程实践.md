---
title: ping命令执行的过程实践
date: 2023-01-30 15:38:06
---

# ping命令执行的过程实践

> 网络包在物理层传输是根据mac地址进行传输的

## ping 执行的过程

* 根据路由表和目标ip计算走哪个网卡
* 根据网卡的ip和子网掩码计算源ip和目标ip是否在一个子网
	* 源ip和目标ip在一个子网
	查询arp缓存中是否存在目标ip的mac地址
		* 不存在
		给整个子网的所有机器广播发送一个 arp查询，并缓存结果
		* 存在
		
	把ping命令封成一个icmp包，填上包头（包括IP、mac地址）
	* 源ip和目标ip不在同一个子网
	通过网关进行转发
	网关查看目标ip是否是自己的
		* 不是
		重复以上步骤
		* 是
		处理请求响应信息

整个过程mac地址一直在变，源ip和目标ip不变



### 源ip和目标ip不在一个子网
```
[root@iZ8vba21wy1jbsrm04fdrbZ ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.17.111.253  0.0.0.0         UG    0      0        0 eth0
10.244.0.0      10.244.0.0      255.255.255.0   UG    0      0        0 flannel.1
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
172.17.96.0     0.0.0.0         255.255.240.0   U     0      0        0 eth0
172.18.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
[root@iZ8vba21wy1jbsrm04fdrbZ ~]# arp -a | grep 172.17.111.253
gateway (172.17.111.253) at ee:ff:ff:ff:ff:ff [ether] on eth0
[root@iZ8vba21wy1jbsrm04fdrbZ ~]# tcpdump -netvv -i eth0 icmp
tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
00:16:3e:09:4f:29 > ee:ff:ff:ff:ff:ff, ethertype IPv4 (0x0800), length 98: (tos 0x0, ttl 64, id 37748, offset 0, flags [DF], proto ICMP (1), length 84)
    172.17.99.209 > 110.242.68.4: ICMP echo request, id 27090, seq 1, length 64
ee:ff:ff:ff:ff:ff > 00:16:3e:09:4f:29, ethertype IPv4 (0x0800), length 98: (tos 0x14, ttl 50, id 37748, offset 0, flags [DF], proto ICMP (1), length 84)
    110.242.68.4 > 172.17.99.209: ICMP echo reply, id 27090, seq 1, length 64
```

### 源ip和目标ip在同一个子网
```
[root@iZ8vba21wy1jbsrm04fdrbZ ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.17.111.253  0.0.0.0         UG    0      0        0 eth0
10.244.0.0      10.244.0.0      255.255.255.0   UG    0      0        0 flannel.1
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
172.17.96.0     0.0.0.0         255.255.240.0   U     0      0        0 eth0
172.18.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
[root@iZ8vba21wy1jbsrm04fdrbZ ~]# arp -a | grep 172.17.99.208
? (172.17.99.208) at ee:ff:ff:ff:ff:ff [ether] on eth0
[root@iZ8vba21wy1jbsrm04fdrbZ ~]# tcpdump -netvv -i eth0 icmp
tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
00:16:3e:09:4f:29 > ee:ff:ff:ff:ff:ff, ethertype IPv4 (0x0800), length 98: (tos 0x0, ttl 64, id 32302, offset 0, flags [DF], proto ICMP (1), length 84)
    172.17.99.209 > 172.17.99.208: ICMP echo request, id 2417, seq 1, length 64
ee:ff:ff:ff:ff:ff > 00:16:3e:09:4f:29, ethertype IPv4 (0x0800), length 98: (tos 0x0, ttl 64, id 35464, offset 0, flags [none], proto ICMP (1), length 84)
    172.17.99.208 > 172.17.99.209: ICMP echo reply, id 2417, seq 1, length 64
```

