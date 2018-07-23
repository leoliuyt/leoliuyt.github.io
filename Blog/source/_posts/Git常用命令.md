---
title: Git常用命令
date: 2018-05-18 21:03:49
tags: git
categories: tool
---
## 配置命令

```
git config
```

## 操作命令

```
//由工作区提交到暂存区
git add

//由暂存区提交到仓库
git commit -m '提交记录'

//将已追踪过的文件自动暂存，直接提交
git commit -a 

//重新提交：漏掉了几个文件没有添加，或者提交信息写错了
git commit --amend

//比较的是工作区与暂存区的不同
git diff

//比较的是暂存区的变化
git diff --staged //(或者是--cache)

//移除文件（已追踪过的文件）连同工作区的文件一同删除
git rm readme.md

//如果要移除的文件被修改过
git rm -f readme.md

//只移除暂存区中的文件,保留工作区的
git rm --cache readme.md

//查看提交历史
git log

//用来显示每次提交的内容差异(-2 代表显示最近的2次)
git log -p -2

//取消工作区的修改[危险]
git checkout -- readme.md

//取消暂存区的修改
git reset HEAD readme.md

//查看分支
git branch

//查看每一个分支的最后一次提交
git branch -v

//查看 已经合并到当前分支的分支
git branch --merged 

//查看 尚未合并到当前分支的分支
git branch --no-merged

//删除分支
git branch -d dev

//rebase 
先切到test1分支
git checkout test1
再执行rebase
git rebase dev
再切换到dev分支
git checkout dev
最后执行merge
git merge test1

```

## 远程仓库操作

```
//查看远程仓库信息
git remote -v

//添加远程仓库
git remote add <shortname> <url>

//从远程仓库拉取数据,执行完成后，将会拥有那个远程仓库中所有分支的引用，可以随时合并或查看。
git fetch [remote-name]

//推送本地分支到远程仓库
git push origin dev

//删除远程分支
git push origin --delete dev

//从服务器上抓取本地没有的数据,并不会修改工作目录中的内容
git fetch
```