---
title: pixel5刷机
date: 2023-03-27 15:38:06
---

# pixel5刷机配置

以window11+pixle5为例，其它的手机可以根据需要自动调整
驱动win11会自动安装

## 环境准备

* window/mac/linux
* type-c数据线（自带）

## 刷机前置准备

### adb工具安装

根据不同的平台选择不同的二进制文件，可以加到环境变量种也可以不加。[下载地址](https://developer.android.com/studio/releases/platform-tools)

### 下载

根据自己的手机型号下载对应的版本，不同的手机厂商可再官网下载rom或者从XDA论坛找适用第三方rom，[pixle手机rom下载地址](https://developers.google.com/android/images)
下载最新的magisk apk文件，[下载地址](https://github.com/topjohnwu/Magisk/releases)

## 刷机步骤
* 开发者模式下开启oem lock,数据线连接到手机，使用命令检验是否连接成功`adb devices -l`
* 开机状态下，打开开发者模式并允许adb调试，使用adb工具`adb reboot bootloader`；关机状态下载长按开机键+音量下键，不同的手机不一样自行查询
* 解锁bootloader，执行`fastboot flashing unlock`
* (可选)清空用户数据`fastboot -w`,一般刷第三方rom需要执行此步骤
* 如果没有将adb添加到环境变量可以临时设置当前shell的变量`set PATH=%CD%;%PATH%`,需要进入到包含adb可执行文件的目录
* 解压下载的pixel rom，进入目录，执行命令`flash-all`，等刷写完成
* 重启并安装magisk
* 使用官方安装教程安装设置magisk，[教程地址](https://topjohnwu.github.io/Magisk/install.html)


## 刷机后设置

### 安装模块safetynet-fix
* 使用magisk安装[safetynet-fix](https://github.com/kdrag0n/safetynet-fix/releases)，以通过谷歌的safetynet验证，很多银行、金融、支付类的app需要校验。
* 在Magisk设置种开启Zygisk\Enforce DenyList,可以在Configure DenyList种添加要隐藏root的app


### 安装VoLTE VoWiFi 5G模块

* [GitHub地址](https://github.com/stanislawrogasik/Pixel5-VoLTE-VoWiFi),支持在不同的国家不同的地址开启volte、vowifi以及5g支持











