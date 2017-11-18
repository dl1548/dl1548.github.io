---
title: django时间问题
date: 2017-05-19 21:24:29
tags: django
categories:
    - web
    - django
copyright: true
---

配置文件settings.py中，有两个配置参数是跟时间与时区有关的
> TIME_ZONE  和 USE_TZ

如果`USE_TZ`设置为`True`时，
使用系统默认设置的时区`America/Chicago`
此时的TIME_ZONE不起任何作用

如果`USE_TZ` 设置为`False`
`TIME_ZONE`设置为`None`，使用默认的`America/Chicago`时间。
`TIME_ZONE`设置为其它时区的话，

Windows系统，则`TIME_ZONE`无用，Django使用本机的时间。

如果为其他系统，以设置为准,上海的UTC时间 `USE_TZ = False, TIME_ZONE = 'Asia/Shanghai'`

___
