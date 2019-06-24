---
title: django RESTful API
date: 2018-10-21 19:50:05
tags: django
categories:
    - web
    - django
copyright: true
password: woshizhu
---

[django RESTful API 官网](https://www.django-rest-framework.org/)
开发中经常遇到前后端分离,实现就要借助API.
API简单说就是开发人员提供编程接口被其他人调用,他们调用之后会返回数据供其使用
API的类型有多种，但是现在比较主流且实用的就是RESTful API.
本文用作个人记录
<img src="/images/drf-api-docs.png" alt="drf-api-docs" align=center />
<!-- more -->

#### 依赖环境

```
Django (1.11.11)
django-filter (1.1.0)
djangorestframework (3.8.2)
```



我的项目如下,`myproject`是项目 , `api`应用,`data`是主应用

```
├── myproject
│   ├── wsgi.py
│   ├── urls.py
│   ├── settings.py
│   └── __init__.py
├── manage.py
└── api
    ├── views.py
    ├── urls.py
    ├── tests.py
    ├── serializer.py
    ├── models.py
    ├── migrations
    │   └── __init__.py
    ├── __init__.py
    ├── apps.py
    └── admin.py
└── data
    ├── views.py
    ├── tests.py
    ├── models.py
    ├── migrations
    │   └── __init__.py
    ├── __init__.py
    ├── apps.py
    └── admin.py
```
#### 注入应用

修改`setting`文件

```
INSTALLED_APPS = [
    ...
    'data',
    'rest_framework',
    'api',
]
```

##### 全局渲染

其实渲染有多种类型,局部的全局的等

这里设置的是全局的

在项目`setting` 追加

```
REST_FRAMEWORK = {
    'DEFAULT_RENDERER_CLASSES': (
        'rest_framework.renderers.JSONRenderer',
    )
}
```





#### 使用模型序列器

ModelSerializers

`api`项目下新建文件`serializer.py`和`urls.py`

内如参考如下


##### serializer.py

```
from rest_framework import serializers
from data.models import * #导入data下的models

class OsInfoSerializer(serializers.ModelSerializer):
    class Meta:
        model =OsInfo
        #fields = '__all__'  # 所有
        fields =('id','host_ip','os_sys','os_version',等等)

#定义了针对数据库表数据的过滤筛选类
```

##### urls.py

```
from django.conf.urls import url, include
from rest_framework.routers import DefaultRouter
from api import views as api_views

router = DefaultRouter()

# os 
router.register('osinfo/one',api_views.OsInfoOne,base_name='os_info_one')

urlpatterns = [
    url(r'^', include(router.urls)),
]
```
##### views.py(api下)

```
# data db
from data.models import *
#serializer
from serializer import * 
#rest framework
import django_filters
from rest_framework import viewsets, filters
from rest_framework.response import Response

class OsInfoOne(viewsets.ViewSet):
    def list(self,request):
        ip = request.GET['ip']
        if ip is not None:
            try:
                queryset = OsInfo.objects.filter(host_ip=ip)
                serializer_class = OsInfoSerializer

                serializer = OsInfoSerializer(queryset, many=True)
            except:
                return Response('OsInfoOne serializer error')
        else:
            return Response('missing parameter ip=xxx')

        if serializer.data == []:
            return Response('None')
        else:
            return Response(serializer.data)
```
##### urls.py(project下)

```
urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^api/', include('api.urls')),
    ....
```

到这里整个流程基本走完了.

运行程序
```
python manage.py runserver 0.0.0.0:8888
```
浏览器访问
[http://ipaddress:8888/api/osinfo/one/?ip=192.168.1.55](http://ipaddress:8888/api/osinfo/one/?ip=192.168.1.55)

返回结果:

```
[
  {
    id: 30,
    host_ip: "192.168.1.55",
    os_sys: "GNU/Linux",
    os_version: "CentOS Linux release 7.2.1511 (Core) ",
    os_kernel: "3.10.0-327.el7.x86_64",
    cpu_model: " Intel(R) Xeon(R) CPU E5-2643 v2 @ 3.50GHz",
    cpu_num: "2",
    cpu_core: " 1",
    mem_total: "4096",
    disk_total: "100 GB",
    product_id: " VMware-42 1a 0e 57 54 30 bc a6-67 a5 f7 63 41 dd ed db"
  }
]
```

[更多方法的使用见官网或参考这里](https://blog.csdn.net/ppppfly/article/details/51077433#t7)

---

