---
title: Django连接MySql
date: 2017-05-13 14:51:07
tags:
    - django
    - mysql
categories:
    - web
    - django
copyright: true
---
**前言**
Django默认连接的是sqlite,简单的开发测试还不错,随着需求的增加,难免需要更换

<!--more-->
#### setting.py
```
DATABASES = {
    'default': {
#        'ENGINE': 'django.db.backends.sqlite3',
#        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
         'ENGINE': 'django.db.backends.mysql',
         #数据库名字
         'NAME': 'study', 
         'USER': 'root',
         'PASSWORD': 'centos',
         'HOST': '127.0.0.1',
         'PORT': '3306',
         'OPTIONS': {
            'autocommit': True,
            'init_command': "SET sql_mode='STRICT_TRANS_TABLES'",
        },
    }
}

```


#### 安装相关模块
> python3 不在支持MySQLdb
>安装pymsql  `pip3 install pymysl`

```
#去项目下修改__init__.py,使其默认数据库模块为pymysql
import pymysql
pymysql.install_as_MySQLdb()
```

#### 项目下执行
```
python manage.py makemigrations
python manage.py migrate
```


#### 操纵models.py
**数据库相关一般都写在这个模块下** 前提，应用要加到setting中
```
from django.db import models

# Create your models here.
#新建了一个Student类，继承自models.Model
class Student(models.Model):
    name = models.CharField(max_length=128)
    age = models.IntegerField(max_length=3



#上面代码其实就相当于原生sql

CREATE TABLE myapp_person (
    "id" serial NOT NULL PRIMARY KEY,
    "name" varchar(30) NOT NULL,
    "age" int() NOT NULL
);

#然后进行同步
python manage.py makemigrations
python manage.py migrate

#生成一个
项目名称+下划线+小写类名的表
比如项目叫study_1，那表名就叫study_1_student
```

```
#插入数据
class Student(models.Model):
        Student.objects.create(name='lizili',age=18)
        Student.objects.create(name='vic',age=22)
        Student.objects.create(name='zhang',age=12)

#返回数据
    def __str__(self):  #固定格式
        return u'name: %s , age:%s' % (self.name,self.age)
```
```
#需要数据的页面
#导入
from study_1.models import Student
#查询，展示
def test(request):
    student_list = Student.objects.all()
    student_str = map(str,student_list)
    return HttpResponse(student_str)
```
