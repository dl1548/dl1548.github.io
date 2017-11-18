---
title: Django静态文件
date: 2017-05-17 12:15:46
tags:
    - django
categories:
    - web
    - django
copyright: true
---

Django静态文件配置
<!--more-->

Django 1.10
#### 目录结构
```
Project/
├── db.sqlite3
├── manage.py
├── Project
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
├── app01
│   ├── admin.py
│   ├── apps.py
│   ├── __init__.py
│   ├── migrations
│   │   ├── __init__.py
│   ├── models.py
│   ├── static  #静态文件位置
│   │   └── css.css
│   ├── templates  #网页模版位置
│   │   └── index.html
│   ├── tests.py
│   ├── views.py
└── templates
```

#### settings修改(项目下)
```
在STATIC_URL = '/static/'  后添加
STATIC_ROOT = os.path.join(BASE_DIR, 'static')
#如若不行，添加如下信息
STATICFILES_DIRS = (
     os.path.join(BASE_DIR, '/home/ubuntu/django/wechat/zbx/static'),     #路径          
)

```
#### urls修改(项目下)

```
#导入库文件
from django.conf import settings
from django.conf.urls.static import static
末尾追加
+ static(settings.STATIC_URL, document_root = settings.STATIC_ROOT)

#最后类似

from django.conf import settings
from django.conf.urls.static import static
urlpatterns = [
    url(r'^admin/', admin.site.urls),
] + static(settings.STATIC_URL, document_root = settings.STATIC_ROOT)

```

#### 网页模版下引用
```
<link rel="stylesheet" href="/static/css.css">
```
重新运行项目即可使用了。
