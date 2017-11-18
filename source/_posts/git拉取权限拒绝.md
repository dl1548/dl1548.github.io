---
title: git拉取权限拒绝
date: 2017-08-3 8:18:35
tags:
    - git
categories:
    - 运维工具
    - git
copyright: true
---
自己挖的坑...
<!--more-->
由于昨晚在家新增和修改了文件,今早到公司就进行了个pull的操作
```
来自 git.oschina.net:user/django
 * branch            master     -> FETCH_HEAD
更新 c1dc841..f21e3fe
error: unable to unlink old 'study/app1/__pycache__/__init__.cpython-35.pyc' (权限不够)
```

全是权限不足,第一反应是sudo的原因.没多想就直接sudo操作了,然后提示

```
Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```
搜索

解决办法都是说重新生成秘钥.然后复制到git.
但是自己的密钥并没有变更过,以防万一,还是操作了一遍.

并没有解决问题.

想想如果不是远程的问题,那就是本地的了.

原来昨晚创建了新的文件夹用的是sudo创建的...

更改文件夹所属就好了
```
chown user:group /dir
```
