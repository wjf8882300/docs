---
title: hexo搭建github博客
date: 2021-12-21 19:43:24
tags: git
categories: 工具
---
1）GitHub上面新建一个repository

2）clone到本地

3）安装hexo
npm i hexo-cli -g

4）验证是否安装成功
hexo -v

5）找个空目录初始化
hexo init

6）生成静态页面
hexo g

7）打开本地网页：http://localhost:4000
hexo s

8）修改配置文件_config.yml
deploy:
  type: git
  repo: git@github.com:wjf8882300/docs.git
  branch: master

9）安装git插件
npm install --save hexo-deployer-git

10）把生成的文件拷贝到git目录

11）git提交
hexo d

12）github选择repository——>Settings——>Pages，选择分支和主题。上面就是博客地址。

13）写文章
hexo new post "docker"