---
title: git常用命令及操作笔记
date: 2019-10-01 18:25:02
tags:
    - git
categories:
    - 工具使用
summary: 记录个人常用到的git命令及场景操作，用于快速翻看。
---
## 场景操作指北
### 删除commit历史记录
1. 新建无`commit`历史的分支
``` bash
$ git checkout --orphan latest_branch
```
2. 添加所有文件
``` bash
$ git add -A
```
3. 提交变更
``` bash
$ git commit -am "commit message"
```
4. 删除`master`分支
``` bash
$ git branch -D master
```
5. 讲当前分支重命名为`master`分支
``` bash
$ git branch -m master
``` 
6. 最后推送到远程仓库
``` bash
$ git push -f origin master
``` 