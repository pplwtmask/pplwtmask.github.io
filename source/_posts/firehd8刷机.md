---
title: fire hd8 2018 降级、ROOT、刷机硬核教程
date: 2020-05-21 15:38:06
---

### 刷机步骤
**环境准备：** 
* linux（最好是Ubuntu，可以装在优盘）
>安装 python3, PySerial, adb and fastboot. 对于Debian/Ubuntu系统，可以执行命令：`sudo apt install python3 python3-serial android-tools-adb android-tools-fastboot`

* 铜线（操作方便）或其它导电体
* 塑料翘片（打开平板后盖，其实很简单！）
* 数据线（最好是平板自带的数据线）

**附件目录：**
* amonet.tar.gz
* 6300.zip
* Magisk-v18.0.zip
* finalize.zip

文件下载链接：

**刷机步骤：**
* 将平板关机（未连接电脑），注意如果安装了`ModemManager`要禁用或者卸载。
* 打开平板的后盖，（没有螺丝，是卡扣卡住的，很简单就能打开）
* 打开后看到主板的左边有四个测试点，依次是DAT0, RST, CMD, CLK，只需要将CLK接地即可（可以用铜线或其它方式）。
* 解压文件`amonet.tar.gz`并执行命令`./bootrom-step.sh`,此时窗口将显示"Waiting for the bootrom"
* 此时将接地完成的平板连接电脑，等待窗口显示"* * * Remove the short and press Enter * * * ",如果未出现则重复以上步骤直至出现为止。
* 以上成功执行之后平板处于`fastboot`模式，会看到屏幕有Amazon的logo，此时执行命令`./fastboot-step.sh`
* 以上成功执行之后平板进入`recovery`，如果屏幕是暗的可以按两次开机键点亮屏幕
* 将刷机用到的文件通过adb或者recovery的存储传入手机，adb命令`adb push 6300.zip /sdcard adb push Magisk-v18.0.zip /sdcard   adb push finalize.zip /sdcard`
* 通过recovery刷入6300.zip，三清之后开机，注意不要联网
* 关机，按音量下键+开机键进入recovery，刷入Magisk-v18.0.zip、finalize.zip
