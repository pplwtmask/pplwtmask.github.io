---
title: git使用日志
date: 2021-05-21 15:38:06
---


### git使用记录

#### 一般流程

```bash
git clone git@github.com:michaelliao/learngit.git
git checkout -b dev origin/dev  //git checkout -b <branch> --track <remote>/<branch>
```
```bash
git clone git@github.com:michaelliao/learngit.git
git checkout -b dev //创建并切换到dev分支
git branch -u origin/dev	//设置上游分支
```
```bash
$ git pull origin next
或
$ git fetch origin
$ git merge origin/next
```

**注意：** `git checkout <branch>`没有加`--no-guess`选项时，会先在本地查找当前分支，未找到则会查找远程仓库

#### 修改提交
```
git commit -m 'initial commit'
git add forgotten_file
git commit --amend
```

#### 版本回退
```bash
git log --oneline
git reset --hard 1094a
git push -f
```

#### 撤销回退
```bash
git reflog
git reset --hard 1094a
git reset --soft 1094a //保留更改
git reset --hard origin/<branch_name> //重置原始版本
```

#### 删除中间某次提交
```bsah
git revert -n 1094a
git commit -m "revert 1094a"
git push
```

#### 隐藏工作区
```bash
git stash
git stash list
git stash pop
```

#### rebase
Reapply commits on top of another base tip
```
# 将最新的三次提交压缩到一次
git rebase -i HEAD~3
# 将当前dev代码rebase到当前分支，删除分支后将是一条直线
git rebase dev
```

#### 添加远程仓库
```bash
git remote add origin git@github.com:michaelliao/learngit.git
git push -u origin master
```

#### 创建删除远程分支
```bash
git push origin serverfix:serverfix
git push origin --delete serverfix
```

#### 合并两个无关联的仓库
```
$git pull origin master --allow-unrelated-histories
```

#### 配置别名
全局配置文件存放在`~/.gitconfig`
将日志打印的优雅：
```bash
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

**git文档地址：https://www.git-scm.com/book/zh/v2**

