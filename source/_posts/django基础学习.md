---
title: Django基础
date: 2017-03-19 22:42:57
tags: django
categories:
    - web
    - django
copyright: true
---


<img src="/images/django.jpg" width = "50%" height = "30%" alt="django" align=center />
#### 前言
感谢[自强学堂](http://www.ziqiangxuetang.com)
<!-- more -->



#### Django简介

  **urls.py**

>   网址入口，关联到对应的view.py中的一个函数（或者generic类），访问网址就对应一个函数

 **view.py** 

>   处理用户发出的请求，从urls.py中对应过来，通过选人templates中的网页，可将内容显示，比如，登陆后的用户名，用户请求的数据，输出到网页

 **models.py**

>   与数据库关联，存入或读取数据的时候会用到这个

 **forms.py** 

>   表单，用户在浏览器上输入数据提交，对数据的验证工作以及输入框的生成等工作



**templates**文件夹

>   views.py 中的函数渲染templates中的Html模板，得到动态内容的网页，当然可以用缓存来提高速度。

 **admin.py**

>   后台，可以用很少的代码量，拥有一个强大的后台

 **settings.py**

>   Django 的设置，配置文件，比如Debug开关，静态文件位置等



#### Django环境搭建


>   Django 1.5.x 支持 Python 2.6.5 Python 2.7, Python 3.2 和 3.3.
>
>   Django 1.6.x 支持 Python 2.6.X, 2.7.X, 3.2.X 和 3.3.X
>
>   Django 1.7.x 支持 Python 2.7, 3.2, 3.3, 和 3.4 （注意：Python 2.6 不支持了）
>
>   **Django 1.8.x 支持 Python 2.7, 3.2, 3.3, 3.4 和 3.5. （长期支持版本 LTS)**
>
>   Django 1.9.x 支持 Python 2.7, 3.4 和 3.5. 不支持 3.3 了
>
>   Django 1.10.x 支持 Python 2.7, 3.4 和 3.5.
>
>   **Django 1.11.x 下一个长期支持版本，将于2017年4月发布**

  **更详细的可以[参考这里](https://www.djangoproject.com/download/)**一般来说，选择长期支持版本比较好。

>   使用最新版本的问题就是，可能要用到的一些第三方插件没有及时更新，无法正常使用这些三方包。

>   如果是学习，可以选择目前的 Django 1.8.x 来进行，遇到问题也容易找到答案。
>
>   当然如果需要新版本的功能也可以使用新版本，毕竟 Django 1.9 以后admin界面还是更漂亮些


#### 安装Django

安装pip

ubuntu `sudo apt-get install python-pip`

centos `yum -y install python-pip`

升级pip

`pip install --upgrade pip`

利用pip安装Django

```
（sudo) pip install Django
或者 (sudo) pip install Django==1.8.16 或者 pip install Django==1.10.3
```

#### 搭建多个互不干扰的开发环境

```
# 安装:
(sudo) pip install virtualenv
```

```shell
mkdir myproject
cd myproject

virtualenv --no-site-packages test

#命令virtualenv就可以创建一个独立的Python运行环境，我们还加上了参数--no-site-packages，这样，已经安装到系统Python环境中的所有第三方包都不会复制过来，这样，我们就得到了一个不带任何第三方包的“干净”的Python运行环境。

source test/bin/activate
#命令行提示符的最前方，会提示当前所在的python环境

#然后就可以在此环境下 进行开发/测试等

deactivate  #退出环境
```

####Django的基本命令（请牢牢记住，不能tab）

##### 新建一个django project

```
django-admin.py startproject zixue
```

##### 新建app

```bash
python manage.py startapp app-name
或 django-admin.py startapp app-name

#一般一个项目有多个app, 当然通用的app也可以在多个项目中使用。
```

##### 同步数据库

```bash
python manage.py syncdb
 
#注意：Django 1.7.1及以上的版本需要用以下命令
python manage.py makemigrations
python manage.py migrate

#这种方法可以创建表，当你在models.py中新增了类时，运行它就可以自动在数据库中创建表了，不用手动创建。

#备注：对已有的 models 进行修改，Django 1.7之前的版本的Django都是无法自动更改表结构的，不过有第三方工具 south,
```

##### 使用开发服务器

开发服务器，即开发时使用，一般修改代码后会自动重启，方便调试和开发，但是由于性能问题，建议只用来测试，不要用在生产环境。

```bash
python manage.py runserver
 
# 当提示端口被占用的时候，可以用其它端口：
python manage.py runserver 8001
python manage.py runserver 9999
（当然也可以kill掉占用端口的进程）
 
# 监听所有可用 ip （电脑可能有一个或多个内网ip，一个或多个外网ip，即有多个ip地址）
python manage.py runserver 0.0.0.0:8000
# 如果是外网或者局域网电脑上可以用其它电脑查看开发服务器
# 访问对应的 ip加端口，比如 http://172.16.20.2:8000
```

##### 清空数据库

```bash
python manage.py flush
```

##### 创建超级管理员

```bash
python manage.py createsuperuser
 
# 按照提示输入用户名和对应的密码就好了邮箱可以留空，用户名和密码必填
 
# 修改 用户密码可以用：
python manage.py changepassword username
```

导入导出数据

```bash
python manage.py dumpdata appname > appname.json
python manage.py loaddata appname.json
```



##### Django羡慕的环境终端

```bash
python manage.py shell
```

##### 数据库命令行

```bash
python manage.py dbshell

Django 会自动进入在settings.py中设置的数据库，如果是 MySQL 或 postgreSQL,会要求输入数据库用户密码。
在这个终端可以执行数据库的SQL语句。如果您对SQL比较熟悉，可能喜欢这种方式。
```

##### 更多命令

```
终端上输入 python manage.py 可以看到详细的列表，在忘记子名称的时候特别有用。
```



#### Django的视图与网址

##### 创建项目

```SHELL
django-admin.py startproject mysite
```

创建成功后，目录如下

```
mysite
├── manage.py
└── mysite
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
    
#新建了一个 mysite 目录，其中还有一个 mysite 目录
#这个子目录 mysite 中是一些项目的设置settings.py 文件
#总的urls配置文件 urls.py 以及部署服务器时用到的 wsgi.py 文件
# __init__.py 是python包的目录结构必须的，与调用有关。
```

##### 新建应用（app） 名字叫learn

```
python manage.py startapp learn   #learn只是一个app的名称
```

**把我们新定义的app加到settings.py中的INSTALL_APPS**

```python
mysite/mysite/settings.py

INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
 
    'learn',
)

#备注,这一步是干什么呢? 新建的 app 如果不加到 INSTALL_APPS 中的话,
#django 就不能自动找到app中的模板文件(app-name/templates/下的文件)
#和静态文件(app-name/static/中的文件) , 后面你会学习到它们分别用来干什么.
```

##### 定义视图函数 （访问页面的内容）

修改 应用 learn 中的view.py

```python
#coding:utf-8
from django.http import HttpResponse

def index(request):
    return HttpResponse(u'欢迎')
    
#第一行是声明编码为utf-8, 因为我们在代码中用到了中文,如果不声明就报错.
#第二行引入HttpResponse，它是用来向网页返回内容的，
#就像Python中的 print 一样，只不过 HttpResponse 是把内容显示到网页上。

#我们定义了一个index()函数，第一个参数必须是 request，与网页发来的请求有关，request
#变量里面包含get或post的内容，用户浏览器，系统等信息#在里面（后面会讲，先了解一下就可以）。
#函数返回了一个 HttpResponse 对象，可以经过一些处理，最终显示几个字到网页上。
#那问题来了，我们访问什么网址才能看到刚才写的这个函数呢？怎么让网址和函数关联起来呢？
```



##### 定义视图函数相关的URL

打开mysite下的urls.py

```python
#Django 1.7

from django.conf.urls import patterns, include, url
from django.contrib import admin
admin.autodiscover()
 
urlpatterns = patterns('',
    url(r'^$', 'learn.views.index'),  # new
    # url(r'^blog/', include('blog.urls')),
 
    url(r'^admin/', include(admin.site.urls)),
)
```



```python
#Django1.8 以上

from django.conf.urls import url
from django.contrib import admin

from learn import views as learn_views  # new
 
urlpatterns = [
    url(r'^$', learn_views.index),  # new
    url(r'^admin/', admin.site.urls),
]
```

```python
#开启python测试
python manage.py runserver
#允许远程访问
python manager.py runserver 0.0.0.0:8000 （指定IP和端口)
```



#### 视图与网址进阶

##### 新建项目

##### 新建应用

同上

##### 修改应用下的views.py

```python
from django.shortcuts import render
from django.http import HttpResponse
 
def add(request):
    a = request.GET['a']
    b = request.GET['b']
    c = int(a)+int(b)
    return HttpResponse(str(c))
    
    #注：request.GET 类似于一个字典，更好的办法是用 request.GET.get('a', 0) 当没有传递 a 的时候默认 a 为 0
```

##### 修改项目下的urls.py

```python
from django.conf.urls import url
from django.contrib import admin

from learn import views as learn_views
from calc import views as calc_views

urlpatterns = [
    url(r'^add/$',calc_views.add,name='add'),
    url(r'^$',learn_views.index),
    url(r'^admin/', admin.site.urls),
]
```



##### 打开网址

IP:8000/add/?a=1&b=2



##### 采用add/2/4的方式

进入cala/views.py 定义函数

```python
def add2(request,a,b):
    c = int(a) + int(b)
    return HttpResponse(str(c))
```

修改项目下的urls.py

Django 1.7.X

```python
    url(r'^add/(\d+)/(\d+)/$', 'calc.views.add2', name='add2'),
```

Django 1.8+

```
    url(r'^add/(\d+)/(\d+)/$', calc_views.add2, name='add2'),
    
    #我们可以看到网址中多了 (\d+), 正则表达式中 \d 代表一个数字，+ 代表一个或多个前面的字符，写在一起 \d+ 就是一个或多个数字，用括号括起来的意思是保存为一个子组（更多知识请参见 Python 正则表达式），每一个子组将作为一个参数，被 views.py 中的对应视图函数接收。
访问 http://127.0.0.1:8000/add/4/5/ 就可以看到和刚才同样的效果
```



#### URL name详解

`url(r'^add/$', calc_views.add,name='add'), 这里的name='add' 是用来干什么的呢？`

**简单的来说，name可以在template,models，views中得到对应的网址，相当于给网址去了名字，只要名字不变，网址变不变都是可以获取到的**

修改calc/views.py (应用calc已经在setting.py中INSTALLED_APPS导入,不然模板是找不到的)

```python
from django.http import HttpResponse
from django.shortcuts import render
 
def index(request):
    return render(request, 'home.html')
...

#render 是渲染模板
```



然后新建**templates**文件夹,在新建home.html

```html
<!DOCTYPE html>
<html>
<head>
    <title>自强学堂</title>
</head>
<body>
<a href="/add/4/5/">计算 4+5</a>
</body>
</html>
```

修改项目中的urls.py

```python
...
from learn import views as learn_views
from calc import views as calc_views

urlpatterns = [
    url(r'^$',calc_views.index,name='home'),
    url(r'^add/$',calc_views.add,name='add'),
    url(r'^add/(\d+)/(\d+)/$',calc_views.add2,name='add2'),
    url(r'^admin/', admin.site.urls),
]
```

然后运行服务,访问页面就会出现  计算4+5  的链接

```
<a href="/add/4/5/">计算 4+5</a>

如果这样写“死网址”，会使得在改了网址（正则）后，模板（template)，视图(views.py，用以用于跳转)，模型(models.py，可以用用于获取对象对应的地址）用了此网址的，都得进行相应的更改，修改的代价很大，一不小心，有的地方没改过来，就不能用了。

那么有没有更优雅的方式来解决这个问题呢？
```

```
不带参数的：
{% url 'name' %}
带参数的：参数可以是变量名
{% url 'name' 参数 %}
 
例如：
<a href="{% url 'add2' 4 5 %}">link</a>
```
当 urls.py 进行更改，前提是不改 name（这个参数设定好后不要轻易改），获取的网址也会动态地跟着变，比如改成：

```python
url(r'^new_add/(\d+)/(\d+)/$', calc_views.add2, name='add2')

#add 变成了 new_add，但是后面的 name='add2' 没改，这时 {% url 'add2' 4 5 %} 就会渲染对应的网址成 /new_add/4/5/
```





另外，比如用户收藏夹中收藏的URL是旧的，如何让以前的 /add/3/4/自动跳转到现在新的网址呢？

**要知道Django不会帮你做这个，这个需要自己来写一个跳转方法**：

具体思路是，在 views.py 写一个跳转的函数：

```python
from django.http import HttpResponseRedirect
from django.core.urlresolvers import reverse  # django 1.4.x - django 1.10.x
#  from django.urls import reverse  # new in django 1.10.x
 
def old_add2_redirect(request, a, b):
    return HttpResponseRedirect(
        reverse('add2', args=(a, b))
    )
```

urls.py中

```python
url(r'^add/(\d+)/(\d+)/$', calc_views.old_add2_redirect),
url(r'^new_add/(\d+)/(\d+)/$', calc_views.add2, name='add2'),
```

这样，**假如用户收藏夹中**有 /add/4/5/ ，访问时就会自动跳转到新的 /new_add/4/5/ 了



#### templates

创建一个项目和app

```python
django-admin.py startproject zqxt_tmpl
cd zqxt_tmpl
python manage.py startapp learn
```

把 learn 加入到 settings.INSTALLED_APPS中

```python
INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
 
    'learn',
)
```

打开 learn/views.py 写一个首页的视图

```python
from django.shortcuts import render
 
def home(request):
    return render(request, 'home.html')
```

创建templates/home.html



然后将输入和网址对应

```python
from django.conf.urls import include, url
from django.contrib import admin
from learn import views as learn_views
 
 
urlpatterns = [
    url(r'^$', learn_views.home, name='home'),
    url(r'^admin/', include(admin.site.urls)),
]

#Django1.10+
#url(r'^admin/', admin.site.urls),  #include
```

##### 项目中有多个应用,所以就会有多个templates以及多个index.html

默认情况下Django是不会去区分的,最先找到那个就显示那个,所以如果为了区分的话

就在各个应用的templates下进行在划分

```
project
├── app1
|....
│   ├── templates
│   │   └── app1
│   │       ├── index.html
│   │       └── search.html
├── app2
|.....
│   ├── templates
│   │   └── app2
│   │       ├── index.html
│   │       └── poll.html
```



##### 模版补充知识点

网站模板的设计，一般的，我们做网站有一些通用的部分，比如**导航，底部，访问统计代码等等**

>   可以写一个 base.html 来包含这些通用文件（include)

```html
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}默认标题{% endblock %} - test</title>
</head>
<body>
 
{% include 'nav.html' %}
 
{% block content %}
<div>这里是默认内容，所有继承自这个模板的，如果不覆盖就显示这里的默认内容。</div>
{% endblock %}
 
{% include 'bottom.html' %}
 
{% include 'tongji.html' %}
 
</body>
</html>


#如果需要，写足够多的 block 以便继承的模板可以重写该部分
#include 是包含其它文件的内容，就是把一些网页共用的部分拿出来，重复利用
#改动的时候也方便一些，还可以把广告代码放在一个单独的html中，改动也方便一些，
#在用到的地方include进去。其它的页面继承自 base.html 就好了，
#继承后的模板也可以在 block 块中 include 其它的模板文件。
```

#### templatesde 进阶

 **主要讲解模板中的循环，条件判断，常用的标签，过滤器的使用**

##### 基本字节的显示

views.py

```PYTHON
# -*- coding: utf-8 -*-
from django.shortcuts import render
 
def home(request):
    string = u"学习Django，用它来建网站"
    return render(request, 'home.html', {'string': string})

#在视图中我们传递了一个字符串名称是 string(引号内的表示可悲html调用的) 到模板 home.html
```
home.html

```python
{{ string }}
```
##### for循环

views.py

```PYTHON
def home(request):
    TutorialList = ["HTML", "CSS", "jQuery", "Python", "Django"]
    return render(request, 'home.html', {'TutorialList': TutorialList})
```

home.html

```PYTHON
{% for i in TutorialList %}
{{ i }}
{% endfor %}
```

```

>   for循环要有个结束标记
>
>   一般的变量声明使用{{}}
>
>   功能性的 比如循环，条件判断等使用{%  %}



##### 显示字典的内容

views.py

```python
def home(request):
    info_dict = {'site': u'学堂', 'content': u'IT技术教程'}
    return render(request, 'home.html', {'info_dict': info_dict})
```

home.html

```DJANGO
站点：{{ info_dict.site }} 内容：{{ info_dict.content }}
```

遍历字典

```django
{% for key, value in info_dict.items %}
    {{ key }}: {{ value }}
{% endfor %}
```



##### 条件判断和 for 循环

views.py

```python
def home(request):
    List = map(str, range(100))# 一个长度为100的 List
    return render(request, 'home.html', {'List': List})
```

home.html

```django
{% for item in List %}
    {{ item }}, #每次结束都会在值得后面添加最后，最后一个依然会添加 99,
{% endfor %}    
```

所以，如何判读是不是最后一次遍历呢

>   forloop.last变量  判断是否为最后一项，如果是则为真，反之。

```django
{% for item in List %}
    {{ item }}{% if not forloop.last %}, {% endif %} ##判断不是最后一个则加逗号
{% endfor %}    
```

for循环的其他变量

| 变量                  | 描述                            |
| ------------------- | ----------------------------- |
| forloop.counter     | 索引从1开始计算                      |
| forloop.counter0    | 索引从0开始计算                      |
| forloop.revcounter  | 索引最大长度到1                      |
| forloop.revcounter0 | 索引最大长度到0                      |
| forloop.first       | 遍历元素为第一项时，返回真                 |
| forloop.last        | 遍历元素为最后一项，返回真                 |
| forloop.parentloop  | 用在嵌套for循环中，获取上一层for循环的forloop |

**当列表可能为空的时候用 for  empty**

```DJANGO
<ul>
{% for athlete in athlete_list %}
    <li>{{ athlete.name }}</li>
{% empty %}
    <li>抱歉，列表为空</li>
{% endfor %}
</ul>
```

##### 模板上得到视图对应的网址

```
# views.py
def add(request, a, b):
    c = int(a) + int(b)
    return HttpResponse(str(c))

# urls.py
urlpatterns = patterns('',
    url(r'^add/(\d+)/(\d+)/$', 'app.views.add', name='add'),
)
 
# template html
{% url 'add' 4 5 %}

#name 的方便之处。
#当urls文件发生改变的生后，并不需要去修改html
#因为html中我们使用的是name   也就是add
```

还可以使用 as 语句将内容取别名（相当于定义一个变量）

```
{% url 'some-url-name' arg arg2 as the_url %}
 
<a href="{{ the_url }}">链接到：{{ the_url }}</a>
```

##### 模板中使用逻辑操作

==, !=, >=, <=, >, < 这些比较都可以在模板中使用

```django
{% if var >= 90 %}
成绩优秀，自强学堂你没少去吧！学得不错
{% elif var >= 80 %}
成绩良好
{% elif var >= 70 %}
成绩一般
{% elif var >= 60 %}
需要努力
{% else %}
不及格啊，大哥！多去自强学堂学习啊！
{% endif %}
```

and, or, not, in, not in 也可以在模板中使用

```DJANGO
{% if num <= 100 and num >= 0 %}
num在0到100之间
{% else %}
数值不在范围之内！
{% endif %}
```

判断是否在某个列表中

```
{% if 'ZILI' in List %}
自强学堂在名单中
{% endif %}
```

##### 模板中获取当前网址，当前用户名等

**修改setting**

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                ...
                'django.template.context_processors.request',
                ...
            ],
        },
    },
]
```

然后在模板中就可以调用了

获取用户

```django
{{ request.user }}
```

判断登录

```django
{% if request.user.is_authenticated %}
    {{ request.user.username }}，您好！
{% else %}
    请登陆，这里放登陆链接
{% endif %}
```

获取网址

```django
{{ request.path}}
```

获取当前get 参数

```django
{{ request.GET.urlencode}}
```

**合并到一起举例**

```
<a href="{{ request.path }}?{{ request.GET.urlencode }}&delete=1">当前网址加参数 delete</a>
```

#### 模型（数据库）

>    Django模型适合数据库相关的，与数据库相关的代码一般写在models.py中，Django支持  sqlite3，Mysql，PostgreSQL等数据库，只要在setting.py中进行配置即可

```
django-admin.py startproject learn_models # 新建一个项目
cd learn_models # 进入到该项目的文件夹
django-admin.py startapp people # 新建一个 people 应用（app)
```

**一个项目包含多个应用，一个应用也可以在多个项目中**

>    添加新的项目到settings.py -- INSTALLED_APPS下

##### 修改models.py

与数据库相关的代码一般写在models.py中

```python
from django.db import models
 
class Person(models.Model):
    name = models.CharField(max_length=30)
    age = models.IntegerField()
    
#新建了一个Person类，继承自models.Model, 一个人有姓名和年龄。这里用到了Field，


#上面代码其实就相当于原生sql

CREATE TABLE myapp_person (
    "id" serial NOT NULL PRIMARY KEY,
    "name" varchar(30) NOT NULL,
    "age" int() NOT NULL
);
```

>   表名person_person由Django自动生成：**项目名称+下划线+小写类名**

##### 同步数据库

（默认使用SQLite3.0  无需配置）

```python
Django 1.9 默认使用
python manage.py makemigrations
python manage.py migrate
#同步数据库 migrate代替老版本的syscdb
#这两行命令会对models.py 进行检测，自动发现需要更改的，应用到数据库中去。
127.0.0.1:8000/admin就可以看到简易的CMS系统了
```

**同步数据库命令返回值**

```BASH
[root/myProject/learn_models] ]$python manage.py makemigrations
Migrations for 'people':
  0001_initial.py:
    - Create model Person
[root/myProject/learn_models] ]$python manage.py migrate
Operations to perform:
  Apply all migrations: people, sessions, auth, contenttypes, admin
Running migrations:
  Rendering model states... DONE
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying people.0001_initial... OK
  Applying sessions.0001_initial... OK
[root/myProject/learn_models] ]$


#创建superuser
python manage.py createsuperuser
Username (leave blank to use 'root'): 
Email address: ****@qq.com
Password: 
Password (again): 
Superuser created successfully.

#此时我们登录 ipaddress:8000/admin  就能看到简单的CMS
```

##### Django shell操作数据表

>   Django中的交互式shell来进行数据库的增删改查等操作

###### 增加 查询

```shell
python manage.py shell
#数据增加
In [1]: from people.models import Person

In [2]: Person.objects.create(name='lizili',age=18)#新加数据
Out[2]: <Person: Person object>
#数据库查询
#查询所有，返回一个列表，无对象返回空
In [6]: Person.objects.all()
Out[6]: [<Person: Person object>]

#查询指定对象
In [4]: a=Person.objects.get(id=1)
In [17]: a.name
Out[17]: 'lizili'

#每次都要赋值才能查找，很麻烦，所以可以去modules.py中对语句进行修改
from django.db import models
# Create your models here.
class Person(models.Model):
    name =models.CharField(max_length=30)
    age = models.IntegerField()
    def __str__(self):
        return u'name:%s , age:%s' % (self.name,self.age)

#这样每次就可以直接查询了
In [1]: from people.models import Person
In [2]: Person.objects.get(id=1)
Out[2]: <Person: name:lizili , age:26>
```

######  新建数据

```SHELL
#1
Person.objects.create(name=name,age=age)
#2
p = Person(name="WZ", age=23)
p.save()
#3
p = Person(name="TWZ")
p.age = 23
p.save()

#4这种方法是防止重复很好的方法，但速度要相对慢些
#返回一个元组，第一个为Person对象，第二个为True或False, 新建时返回的是True, 已经存在时返回False.
Person.objects.get_or_create(name="WZT", age=23)

```

###### 查询数据

```shell
Person.objects.all()#查询所有
Person.objects.all()[:10] #切片操作，获取10个人，不支持负索引，切片可以节约内存
Person.objects.get(name='lizili') #关键字

#get方法
#get是用来获取一个对象的，如果需要获取满足条件的一些人，就要用到filter

#filter方法
Person.objects.filter(name="abc") # 等于Person.objects.filter(name__exact="abc") 名称严格等于 "abc" 的人
Person.objects.filter(name__iexact="abc") # 名称为 abc 但是不区分大小写，可以找到 ABC, Abc, aBC，这些都符合条件
Person.objects.filter(name__contains="abc") # 名称中包含 "abc"的人
Person.objects.filter(name__icontains="abc") #名称中包含 "abc"，且abc不区分大小写
Person.objects.filter(name__regex="^abc") # 正则表达式查询
Person.objects.filter(name__iregex="^abc")# 正则表达式不区分大小写
#filter是找出满足条件的，当然也有排除符合某条件的
Person.objects.exclude(name__contains="WZ") # 排除包含 WZ 的Person对象
Person.objects.filter(name__contains="abc").exclude(age=23) # 找出名称含有abc, 但是排除年龄是23岁的
```

##### 自定义Field

[更多Field](https://docs.djangoproject.com/en/1.10/ref/models/fields/)

>   以后补

##### 数据表的更改

>   当数据库设计完后，发现不满意，需要更改，添加/删除字段。

Django 1.7+

```SHELL
#直接修改models.py 然后执行以下语句即可
python manage.py makemigrations
python manage.py migrate

#会出现一下提示，选 1
# 1) Provide a one-off default now (will be set on all existing rows)
# 2) Quit, and let me add a default in models.py
#新增了字段，但是原来已经的数据没有这个字段，
#当你这个字段没有默认值，又不能为空的时候它就不知道怎么做
#需要选择 1 来指定一个 “一次性的值” 给已有字段。
```

##### QuerySet API

>   **数据库接口**相关的接口（QuerySet API)
>
>   从数据库中查询出来的结果一般是一个集合，这个集合叫做 QuerySet
