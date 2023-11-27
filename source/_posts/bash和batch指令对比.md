---
title: bash和batch指令对比
date: 2023-11-27 15:38:06
---

## bash和batch指令对比

|功能描述|bash命令|batch命令|
|---|---|---|
|查看当前目录|`pwd`|`cd` 或 `echo %cd%`|
|列出目录内容|`ls`|`dir`|
|更改目录|`cd <directory>`|`cd <directory>`|
|创建新目录|`mkdir <directory>`|`mkdir` 或 `md <directory>`|
|删除文件|`rm <file>`|`del <file>`|
|删除目录|`rmdir <directory>` 或 `rm -r <directory>`|`rmdir <directory>` 或 `rd <directory>`|
|复制文件|`cp <source> <destination>`|`copy <source> <destination>`|
|移动或重命名文件|`mv <source> <destination>`|`move <source> <destination>` 或 `ren <source> <new name>`|
|设置环境变量|`export VAR=value`|`set VAR=value`|
|执行另一个脚本或程序|`./<script.sh>`|`<script.bat>` 或 `call <script.bat>`|
|显示文本|`echo <text>`|`echo <text>`|
|附加到文件|`echo <text> >> <file>`|`echo <text> >> <file>`|
|从文件读取|`cat <file>`|`type <file>`|
|查找文本|`grep <pattern> <file>`|`find "<string>" <file>`|
|获取命令输出|`command`  或 `$(command)`|`%command%`|
|条件语句|`if [ condition ]; then ... fi`|`if <condition> ( ... )`|
|循环|`for var in <list>; do ... done`|`for %var in (<list>) do ...`|
|当前用户名|`whoami`|`echo %username%`|
|临时停止脚本|`sleep <seconds>`|`timeout /t <seconds>`|
|文件存在检查|`if [ -f <file> ]; then ... fi`|`if exist <file> ( ... )`|

