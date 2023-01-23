---
title: git使用的一些常用命令
date: 2021-05-21 15:38:06
---


### git使用记录

![](/img/git-conmand.png)

#### 一般流程

```bash
git clone git@github.com:michaelliao/learngit.git
git checkout -b dev origin/dev  //git checkout -b <branch> --track <remote>/<branch>
```
```bash
git clone git@github.com:michaelliao/learngit.git
git checkout -b feature //创建并切换到dev分支
//coding...
git diff //查看硬盘（working tree）和本地仓库的差异
git add . //注意：如果有不想提交的文件不要用`.`
git commit -m 'your descroption'
git push -u origin feature //将当前分支推送到远程并设置上游分支
//master分支有其它人的新的提交
git checkout master
git pull origin master
git checkout feature
git rebase master //以master分支的最新提交为基础，这样提交是一条直线
//maybe have to resovel conflict
git push -f origin feature
//create pull request(gitlab is merge request)
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
//删除某次提交并生成新的提交记录，注意：A分支已经revert某次提交1094a,B分支想重新merge1094a的提交需要删除revert的那次提交
//总之一句话，想要合并已经revert的分支，需要revert你的revert
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
# 将某次提交作为base，并修改之后的提交，可以删除中间的多次提交，压缩等等
# 使用rebase删除会修改rebase某次提交之后的所有提交，相当于是把之后的每次提交重新提交了一次
git rebase -i commit_hash
# 将当前dev代码rebase到当前分支，删除分支后将是一条直线
git rebase dev
```

#### 添加远程仓库
```bash
git remote add origin git@github.com:michaelliao/learngit.git
//合并两个无关联的仓库
git pull origin master --allow-unrelated-histories
git push -u origin master
```

#### 创建删除远程分支
```bash
git push origin serverfix:serverfix
git push origin --delete serverfix
```


#### 配置别名
全局配置文件存放在`~/.gitconfig`
将日志打印的优雅：
```bash
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

**git文档地址：https://www.git-scm.com/book/zh/v2**

