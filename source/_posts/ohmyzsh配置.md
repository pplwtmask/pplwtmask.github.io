---
title: ohmyzsh配置
date: 2022-04-07 15:38:06
---

## 更新
GitHub新的终端美化工具，[oh-my-posh](https://github.com/JanDeDobbeleer/oh-my-posh)

## 环境
* macos

## 查看是否安装zsh
macos默认shell就是zsh，如果不是先[安装zsh](https://github.com/ohmyzsh/ohmyzsh/wiki/Installing-ZSH)
```
$ zsh --version
zsh 5.8 (x86_64-ubuntu-linux-gnu)
```

## 安装ohmyzsh
```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

## 安装代码高亮插件
```
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

```
修改~/.zshrc
plugins=( [plugins...] zsh-syntax-highlighting)


## 安装自动补全插件

```
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```
修改~/.zshrc
plugins=( 
    # other plugins...
    zsh-autosuggestions
)









