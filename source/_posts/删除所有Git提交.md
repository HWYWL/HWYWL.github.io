---
title: 清除某个仓库的所有Git提交，不删仓库
date: 2021-03-05 18:03:27
tags: 
 - git
categories: 
 - git
keywords: "git"
description: 清除某个仓库的所有Git提交，不删仓库
---

### 删除历史记录
```
rm -rf .git
```

### 从当前内容初始化、
```
git init
git add .
git commit -m "初始化提交"
```

### 推送到远程仓库，并覆盖历史数据
```
git remote add origin git@github.com:<YOUR ACCOUNT>/<YOUR REPOS>.git
git push -u --force origin master
```



> 来源：https://gist.github.com/stephenhardy/5470814