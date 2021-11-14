---
title: Git基本命令
categories:
  - 技术大杂烩
tags:
  - Git
date: 2020-12-19 20:30:36
---





# 仓库

```git
git init			# 初始化
```



```git
git clone url			# 下载一个项目
```

# 查看信息



```git
git status			# 查看文件状态
```



# 增加/删除文件

```git
git add 文件名			# 添加指定文件到暂存区
```



```git
git add 文件名			# 添加指定目录到暂存区
```



```git
git add .			# 添加当前目录的所有文件到暂存区
```



# 代码提交





```git
git commit -m '备注'			# 提交暂存区到仓库区
```



```git
git commit 文件名 ... -m '备注'		# 提交暂存区的指定文件到仓库区
```



```git
git push origin 分支名			# 将仓库区的文件推送到线上分支
```



# 分支



```git
git branch			# 列出所有本地分支
```



```git
git branch -r			# 列出所有远程分支
```



```git
git branch -a			# 列出所有本地分支和远程分支
```



```git
git branch 分支名			# 新建一个分支，停留在当前分支
```



```git
git checkout -b 分支名			# 新建一个分支，并切换到该分支
```



```git
git checkout 分支名			# 切换到指定分支，并更新工作区
```



```git
git checkout -				# 切换到上一个分支
```



```git
git merge 分支名			# 合并指定分支到当前分支
```



```git
git branch -d 分支名			# 删除分支
```

