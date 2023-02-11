---
title: ipv6-only服务器使用
date: 2023-02-10 15:38:06
---


# ipv6的vps使用

> 截止到2023/2/11，ipv6还没有普及，但是各大运营商已经支持ipv6，可能只需要在路由器开启ipv6，测试当前网络是否支持ipv6：https://ipv6-test.com/（当前可用不知道以后）


## 连接到服务器

* 如果自己的网络支持ipv6，那可以直接用ssh工具去连接
* 如果自己网络不支持v6,那么可以通过在线ssh工具连接，自行搜索
* 通过v4作为网关，实现端口转发，或者堡垒机等

## 登录之后设置

以乌班图系统为例

* 设置DNS64，这样就能解析ipv4的域名
> `echo -e "nameserver 2a00:1098:2c::1\nnameserver 2a01:4f9:c010:3f02::1\nnameserver 2a00:1098:2b::1\n" > /etc/resolv.conf`
> dns地址https://nat64.net
此时执行 `curl -6 ipinfo.io` 可以看到你的ip信息


## 开启bbr

`wget -O tcp.sh "https://git.io/coolspeeda" && chmod +x tcp.sh && ./tcp.sh`

## ipv6-only变双栈服务器

安装CloudFlare WARP，这样你就能在v6的机器上用v4的网络，感谢CloudFlare

可以通过一下github项目安装

- https://github.com/fscarmen/warp 
- https://github.com/P3TERX/warp.sh
- https://github.com/crazypeace/warp.sh
- https://github.com/ViRb3/wgcf

此时执行`curl -4 ipinfo.io`, 你能看到cf给你分配的ipv4的地址


## 开启tun
TUN或TUNnel与以太网设备非常相似，或者我们可以称之为“虚拟接口”，主要用于某些VPN协议，如OpenVPN，以路由发送和接收网络流量。因此，如果要安装VPN，必须启用TUN。


检查是否开启了TUN，`cat /dev/net/tun`,如果显示cat: /dev/net/tun: No such file or directory， 则表示没开启， 如果显示cat: /dev/net/tun: File descriptor in bad state，则是开启了





























































