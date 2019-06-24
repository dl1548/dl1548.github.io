---
title: django-filter
date: 2019-06-24 08:29:36
tags: django
categories:
    - web
    - django
copyright: true
---



Django-filter是一个通用的、可重用的应用程序来缓解写一些平凡的视图代码。具体地说,它允许用户过滤queryset基于模型的字段，从而显示对应的过滤结果。

使用django-filter 的时候能节省很多查询的后台代码

可自动生成docs文档

<!--more-->


#### 安装

建议在虚拟环境中操作

```
pip  install django-filter
```



#### 导入

```
INSTALLED_APPS = [
	...
	'django_filters',
	'rest_framework',
	...
]
```



#### 使用


##### models.py

用来生成数据库的，生成应用时会自带此文件

```
from django.db import models

class UserInfo(models.Model):
	username = models.CharField(max_length=255)
    phone = models.CharField(max_length=255)
    addr = models.CharField(max_length=255)
```



##### myfilter.py

新建文件，用来定义过滤类

```
from django_filters import rest_framework as filters

class UserInfoFilter(filters.FilterSet):
    """
    获取用户信息
    """
    name = filters.CharFilter(field_name="username", lookup_expr='exact',help_text=u'用户名') # exact 精确匹配
    class Meta:
        model = UserInfo  # models
        fields = ()    # 空=所有  或 (username,)指定列
        
```



##### serializers.py

这个是`DRF`相关的东西，放着供参考

```
from mytest.models import UserInfo
from rest_framework import serializers

class UserInfoSerializerI(serializers.ModelSerializer):
    class Meta:
        model = UserInfo
        fields = '__all__'
```



##### view.py

```
from mytest.models import UserInfo  # 这里导入的是models.py 下的方法
from myfilter import UserInfoFilter
from serializers import UserInfoSerializer

from django_filters.rest_framework import DjangoFilterBackend

# 这里使用了DRF
class FetchUserInof(mixins.ListModelMixin,viewsets.GenericViewSet):
    """
    获取用户信息
    """
    queryset = UserInfo.objects.all()
    serializer_class = UserInfoSerializer   # 这里是DRF相关内容
    filter_backends = (DjangoFilterBackend,)
    filter_class = UserInfoFilter
```

