---
title: git
date: 2021-12-21 18:23:20
tags: git
categories: 工具
---
#### 1.全局配置
```
git config --global user.name "yourname"

git config --global user.email "your@email.com"
```
#### 2.git中添加ssh说明
```生成密钥
ssh-keygen -t rsa -C "wangjingfeng2008@163.com"

把专用密钥添加到 ssh-agent 的高速缓存中：
ssh-add ~/.ssh/id_rsa

如果在使用shh-add的时候提示：
Could not open a connection to your authentication agent.

则需手动开启ssh，如下；
eval `ssh-agent -s`
```
#### 3.删除远程分支
```
git branch -r -d origin/branch-name git push origin :branch-name
```

#### 4.撤销未push的commit
```
git log git reset --hard commit_id
```

#### 5.翻墙
```
git config --global https.proxy http://127.0.0.1:1080

git config --global https.proxy https://127.0.0.1:1080

git config --global --unset http.proxy

git config --global --unset https.proxy npm config delete proxy
```
