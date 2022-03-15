---
title: python安装与pip
date: 2022-03-15 12:57:22
tags: python
categories: 语言
---
### 1.python3安装

从官网下载安装包并点击安装即可。
官网：https://www.python.org/
安装路径：可以自定义安装路径（路径建议为全英文），也可以使用默认的安装路径，默认安装路径一般为C:\Users\shangyj\AppData\Local\Programs\Python\Python37-32

### 2.python环境变量配置
为了保证python和pip调用不受路径限制，需要将python安装路径假如环境变量。
方法：右键点击我的电脑-->属性-->高级系统设置-->高级-->环境变量

在Path变量中新建两项：
python.exe所在路径：C:\Users\shangyj\AppData\Local\Programs\Python\Python37-32\（以实际python的安装路径为准）
python的scripts文件夹路径：C:\Users\shangyj\AppData\Local\Programs\Python\Python37-32\Scripts（以实际scripts路径为准）

> pip3在python的scripts文件夹下面，如果pip3找不到，就是没有配置这个环境变量。

### 3 pip下载依赖包遇到的一些问题

#### 问题1：Could not build wheels for bcrypt which use PEP 517 and cannot be installed directly
```
python -m pip install --upgrade pip -i  https://pypi.tuna.tsinghua.edu.cn/simple
pip3 install --no-use-pep517 bcrypt -i  https://pypi.tuna.tsinghua.edu.cn/simple
```
#### 问题2：the ‘make’ utility is missing the PATH
安装MinGW
- 1）https://cygwin.com/install.html
- 2）安装时注意勾选make、gcc
![选择163的源速度比较快](01.png)	
![勾选gcc，注意后面版本号](02.png)	
![勾选make，注意后面版本号](03.png)	

#### 问题3.Microsoft Visual C++ 14.0 or greater is required.
下载visual studio，并安装，https://visualstudio.microsoft.com/zh-hans/thank-you-downloading-visual-studio/?sku=BuildTools&rel=16
![](04.png)	