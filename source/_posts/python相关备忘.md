---
title: python相关备忘
date: 1111-11-11 11:11:11
tags: 
    - python
categories: python
copyright: true
password: woshizhu
---


> python相关的一些记录,初衷是记录一些 包括web,linux,以及python语法等
<!--more-->

#### web相关

##### python3安装

```python
#安装Python3.4

yum -y install epel-release
yum install python34


#安装pip3
yum -y install python34-pip
或
yum install python34-setuptools
easy_install-3.4 pip

#使用pip3了，如：

pip3 install numpy

#pip更换源安装
阿里云 http://mirrors.aliyun.com/pypi/simple/
中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/ 
豆瓣(douban) http://pypi.douban.com/simple/ 
清华大学 https://pypi.tuna.tsinghua.edu.cn/simple/
中国科学技术大学 http://pypi.mirrors.ustc.edu.cn/simple/
#使用方法很简单，直接 -i 加 url
pip install web.py -i http://pypi.douban.com/simple

```

#### 字符串取数字

```
t = 'aaa18aaa'
print(filter(str.isdigit, t))
```
