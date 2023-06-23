---
title: windows安装TVBOX
date: 2023-06-23 22:38:06
---
# windows安装TVBOX

TVBOX一个开源的电视app，众所周知电视盒子看啥都要vip，有了它配置好数据源可免费看全网视频，开源软件的魅力。但是只有安卓版，安卓平板、安卓手机、智能电视直接安装，windows可以安装模拟器，这里介绍win11自带WSA系统。

## 准备

* win11电脑 
* adb工具（可选）

### adb工具安装

根据不同的平台选择不同的二进制文件，可以加到环境变量种也可以不加。[下载地址](https://developer.android.com/studio/releases/platform-tools)

### TVbox下载
[地址](https://github.com/liu673cn/box)

## 步骤

* 设置->时间和语言->语言和区域，设置成美国，国内不支持
* 开启虚拟化，win+x 任务管理器->性能 可以查看bios是否启用虚拟化，若没启用需要进入bios设置，电脑型号不同设置不同自行查询
* 设置->应用->可选功能->更多winsows功能 开启Hyper-V和虚拟机平台，没有的话就不用设置了
* 打开win商店里的[Windows Subsystem for Android™ with Amazon Appstore](https://www.microsoft.com/en-us/p/windows-subsystem-for-android-with-amazon-appstore/9p3395vx91nr?activetab=pivot:overviewtab)，然后安装
* 安装完会在开始菜单应用中找到WSA，然后开发人员->打开开发人员模式，此时会启动安卓系统并看到连接地址。
* 安装软件，可以通过adb命令安装也可以通过可视化软件安装,自行选择

	1. [Apk File Installer](https://www.microsoft.com/store/productId/9MVVJLDMWPSG)，小白专用
	2. `adb connect 127.0.0.1:port` and `adb install C://path//app.apk`, 需要懂点技术




























