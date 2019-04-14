---
title: django-celery
date: 2018-08-02 08:34:34
tags:
    - django
    - celery
categories:
    - web
    - django
copyright: true
---
Celery 是一个 基于python开发的分布式异步消息任务队列，通过它可以轻松的实现任务的异步处理.执行发生连接中断,自动尝试重新执行.
使用场景:
1: 比如对大量器执行一条批量命令，需要长时间等待的情况下
2: 大量的定时任务,执行相关脚本,检测信息等.
<!--more-->

流程:

任务(异步任务,定时任务) --->消息中间件(broker) --->执行(celery worker)--->存储(backend)

#### 安装

`pip install celery` 即可.

个人环境

```
Django==1.11.11
celery==3.1.26.post2
django-celery==3.2.2
```

celery 默认使用RabbitMQ为broker.当然使用redis也可以.具体支持哪些可以去官网查看
以redis为例,redis安装不赘述,基本开箱即用,活使用docker run 一个.
本地还需要安装celery的redis组件
`pip install -U "celery[redis]"`

配置也很简单
格式为: redis://:password@hostname:port/db_number

参开如下:
```
broker = 'redis://:123qweASD@localhost:6379/0'
backend = 'redis://:123qweASD@localhost:6379/0'
```

无密码格式如下:
redis://localhost:6379/0

#### 单脚本使用

新建文件就叫tasks.py吧

```
#!/usr/bin/python
from celery import Celery

broker = 'redis://:123qweASD@192.168.1.xxx:6379/1'
backend = 'redis://:123qweASD@192.168.1.xxx:6379/2'

app = Celery('tasks', broker=broker, backend=backend)

@app.task
def add(x,y):
    print "running.....",x,y
    return x+y
```

当前目录下执行
`celery -A tasks worker --loglevel=info`
此时celery worker运行,终端可看到类似如下信息

```
...
...
[tasks]
  . tasks.add

[2018-08-02 09:04:26,974: INFO/MainProcess] Connected to redis://:**@192.168.1.244:6379/5
[2018-08-02 09:04:26,980: INFO/MainProcess] mingle: searching for neighbors
[2018-08-02 09:04:28,000: INFO/MainProcess] mingle: all alone
[2018-08-02 09:04:28,019: INFO/MainProcess] celery@zili-test ready.

```

我们在新开个终端,或者同级目录下新建文件就叫get.py吧

```
from tasks import add

res = add.delay(5,11)

# 这里可加上判断
# res.ready() # 任务执行返回true,反之false.
# res.status #执行状态
# 具体的结果方面的方法可在官方查看
# http://docs.celeryproject.org/en/latest/reference/celery.result.html

print res.get()

# 执行可看到返回结果为16

[root@zili-test celery]# python test.py 
16

# 同时,在worker终端可看到如下信息
[2018-08-02 09:10:09,805: WARNING/ForkPoolWorker-1] running.....
[2018-08-02 09:10:09,805: WARNING/ForkPoolWorker-1] 5
[2018-08-02 09:10:09,805: WARNING/ForkPoolWorker-1] 11
[2018-08-02 09:10:09,811: INFO/ForkPoolWorker-1] Task tasks.add[34f964fd-afa9-45d8-845e-4b19c7d19b73] succeeded in 0.00624344032258s: 16

```


#### 项目方式使用

项目结构如下:

```
test
├── celeryconfig.py
├── celery.py
├── __init__.py
├── tasks.py
```

##### celeryconfig.py
celery配置文件,[具体参考官网](http://docs.celeryproject.org/en/latest/userguide/configuration.html#configuration)
```
# 使用Redis作为消息代理
BROKER_URL = 'redis://192.168.1.xxx:6379/1'
 
# 把任务结果保存在Redis中
CELERY_RESULT_BACKEND = 'redis://192.168.1.xxx:6379/2'
 
# 任务序列化和反序列化使用msgpack方案
CELERY_TASK_SERIALIZER = 'msgpack'
 
# 读取任务结果一般性能要求不高，所以使用了可读性更好的JSON
CELERY_RESULT_SERIALIZER = 'json'
 
# 任务过期时间，这样写更加明显
CELERY_TASK_RESULT_EXPIRES = 60 * 60 * 24
 
# 指定接受的内容类型
CELERY_ACCEPT_CONTENT = ['json', 'msgpack']

```

##### celery.py

```
#!/usr/bin/python
from __future__ import absolute_import, unicode_literals

from celery import Celery
#import Celery

broker = 'redis://:123qweASD@192.168.1.244:6379/1'
backend = 'redis://:123qweASD@192.168.1.244:6379/2'

app = Celery('test', broker=broker, backend=backend, include=['test.tasks'])

# Optional configuration, see the application user guide.
app.conf.update(
    result_expires=3600,
)

# app.config_from_object('celery.celeryconfig') 

if __name__ == '__main__':
    app.start()

```

##### tasks.py

```
 
from __future__ import absolute_import
from .celery import app
 
@app.task
def add(x, y):
    return x+y
```

启动 ` celery -A test worker --loglevel=info`
注意test为项目名,启动命令与项目同级

在项目同级下新建脚本调用即可使用celery,注意同级,不要再项目内调用.否则会报错.

```
from test.tasks import add
res = add.delay(5,11)
print res.get()

```

#### 定时任务

celery支持定时任务,设定好任务的执行时间,celery就会定时自动帮你执行,这个定时任务模块叫celery beat.
两种方式 1: 在配置文件中指定  2: 在程序中指定

新建脚本 `crontab_task.py`

`程序中指定`

```
from celery import Celery
from celery.schedules import crontab

broker = 'redis://:123qweASD@192.168.1.244:6379/1'
backend = 'redis://:123qweASD@192.168.1.244:6379/2'

app = Celery('tasks', broker=broker, backend=backend)

@app.on_after_configure.connect
def set_crontab_task(sender, **kwargs):
    sender.add_periodic_task(3.0, test.s('hello'),name='add every 3s')
    sender.add_periodic_task(5.0, test.s('world'), expires=10)
    sender.add_periodic_task(
        crontab(hour=7, minute=30, day_of_week=1),
        test.s('Happy Mondays!'),
    )

#编写test函数.定时任务调用
@app.task
def test(args):
    print args

```


新建脚本 `crontab_task_2.py`

`配置文件中指定`

```
from celery import Celery
from celery.schedules import crontab

broker = 'redis://:123qweASD@192.168.1.244:6379/1'
backend = 'redis://:123qweASD@192.168.1.244:6379/2'

app = Celery('tasks', broker=broker, backend=backend)

@app.task
def send(message):
    print message
    return message


app.conf.beat_schedule = {
    'send-every-10-seconds': {
        'task': 'crontab_task2.send',
        #'schedule': 3.0,
        'schedule': crontab(minute='*/1'),
        'args': ('Hello World',)
    },
}

app.conf.timezone = 'UTC'

```

`crontab` 定时配置示例

|格式|解释|
|---|----|
|crontab()|每分钟执行|
|crontab(minute=0, hour=0)|每天零点|
|crontab(minute=0, hour='*/3')|每三小时执行一次|
|crontab(minute='*/15')|每15分钟执行一次|
|crontab(day_of_week='sunday')|周日开始每分钟执行|
|crontab(<br>minute='*/10',<br>hour='3,17',<br>day_of_week='thu,fri')|每周四周五的3-4,点17-18点,<br>每十分钟执行一次|
|crontab(0, 0,day_of_month='11',month_of_year='5') | 每年的五月十一日|

1: 在脚本同级目录执行`celery -A tasks worker -B` 即启动worker和beat服务
2: 或者先用`celery -A proj worker –loglevel=INFO`启动worker
再用`celery -A tasks beat -s /tmp/celerybeat-schedule`

这里的`-s /tmp/celerybeat-schedule` 指定一个记录文件  


测试脚本
```
from test.tasks import add
res = add.delay(5,11)
print res.get()
```


#### 后台启动

目前我们所有的操作其实都是在shell中前台运行的.关闭终端worker就结束了.
所以我们需要后台的方式来运行程序.目前知道的方法有两个,supervisor和init-script

以init-scritp为例
首先去[celery官方github](https://github.com/celery/celery/tree/3.1/extra/generic-init.d) 下载相关的启动脚本.一个是异步任务,一个是周期任务的

将`celeryd`复制到`/etc/init.d/celeryd` 文件中并赋予可执行的权限
然后在 `/etc/default/` 文件夹下创建 `celeryd` 配置文件：

需要更改`CELERY_BIN` `CELERY_APP` `CELERYD_CHDIR` `CELERYD_USER` `CELERYD_GROUP`
根据实际项目情况更改(用户和组可选择已有的或者新建,注意相关项目权限要给到)

```
# Names of nodes to start
#   most people will only start one node:
CELERYD_NODES="worker1"
#   but you can also start multiple and configure settings
#   for each in CELERYD_OPTS
#CELERYD_NODES="worker1 worker2 worker3"
#   alternatively, you can specify the number of nodes to start:
#CELERYD_NODES=10
# Absolute or relative path to the 'celery' command:
CELERY_BIN="/usr/bin/celery"
#CELERY_BIN="/virtualenvs/def/bin/celery"
# App instance to use
# comment out this line if you don't use an app
CELERY_APP="test"
# or fully qualified:
#CELERY_APP="test.tasks:app"
# Where to chdir at start.
CELERYD_CHDIR="/root/celery"
# Extra command-line arguments to the worker
CELERYD_OPTS="--time-limit=300 --concurrency=8"
# Configure node-specific settings by appending node name to arguments:
#CELERYD_OPTS="--time-limit=300 -c 8 -c:worker2 4 -c:worker3 2 -Ofair:worker1"
# Set logging level to DEBUG
#CELERYD_LOG_LEVEL="DEBUG"
# %n will be replaced with the first part of the nodename.
CELERYD_LOG_FILE="/var/log/celery/%n%I.log"
CELERYD_PID_FILE="/var/run/celery/%n.pid"
# Workers should run as an unprivileged user.
#   You need to create this user manually (or you can choose
#   a user/group combination that already exists (e.g., nobody).
CELERYD_USER="root"
CELERYD_GROUP="root"
# If enabled pid and log directories will be created if missing,
# and owned by the userid/group configured.
CELERY_CREATE_DIRS=1
```

`/etc/init.d/celerybeat {start|stop|restart|create-paths|status}`


如果要运行周期性的任务:

下载`celerybeat` 其余同上,可使用同一个配置文件.

#### django中使用(异步)

Celery 4.0支持django1.8及以上的版本，低于1.8的项目使用Celery 3.1

安装 `django-celery`

##### 项目设置
修改`项目settings.py`

```
import djcelery

djcelery.setup_loader() 
CELERY_TIMEZONE='Asia/Shanghai' #与下面TIME_ZONE应该一致
BROKER_URL='redis://:123qweASD@192.168.1.xxx:6379/1'   #消息队列
CELERY_RESULT_BACKEND = 'redis://:123qweASD@192.168.1.xxx:6379/2' #任务结果
CELERYBEAT_SCHEDULER = 'djcelery.schedulers.DatabaseScheduler'  #使用djcelery默认的数据库调度模型,任务执行周期都被存在你指定的orm数据库中
CELERY_TASK_SERIALIZER = 'json' #任务序列化
CELERY_RESULT_SERIALIZER = 'json' # 结果序列化
CELERY_ACCEPT_CONTENT = ['json'] # 指定接受的内容类型
CELERYD_MAX_TASKS_PER_CHILD = 3 #  每个worker最多执行3个任务就会被销毁，可防止内存泄露
#CELERY_ENABLE_UTC = False

INSTALLED_APPS = [
    ...
    'djcelery',
    ... 
]
```

`settings.py`同级目录下`__init__.py`修改

```
from __future__ import absolute_import

# This will make sure the app is always imported when
# Django starts so that shared_task will use this app.
from .celery import app as celery_app
__all__ = ['celery_app']
```

在项目下新建`celery.py`

```
from __future__ import absolute_import

import os

from celery import Celery

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'zlcelery.settings') #项目视情况

from django.conf import settings

app = Celery('zlcelery') #项目视情况

app.config_from_object('django.conf:settings')

app.autodiscover_tasks(lambda: settings.INSTALLED_APPS)

@app.task(bind=True)
def debug_task(self):
    print('Request: {0!r}'.format(self.request))  #dumps its own request information
```


##### app设置

在需要定时的app下新建`tasks.py`

```
from __future__ import absolute_import

from celery import shared_task
import time
# 这里写定时的方法即可

@shared_task
def add(x, y):
    time.sleep(10)
    return x + y
```

然后在`views.py` 就可以调用此方法了. 

```
def celery_test(request):
    x = request.GET['x']
    y = request.GET['y']

    from .tasks import add   #调用异步方法
    
    #delay函数实现异步
    result = add.delay(x,y)

    res = {'code':200, 'message':'ok', 'data':[{'x':x, 'y':y}]}
    #result = add(2,5)

    #return result
    return render(request, 'zl/return_value.html', {'return_value': json.dumps(res)})
    
    #此时请求可直接返回定义的结果.然后继续下下去.而不需要等待执行结果出来再继续了.
```



###### 同步数据库

```
python manage.py makemigrations

python manage.py migrate
```

###### 创建admin

```
python manage.py createsuperuser

Username (leave blank to use 'work'): admin
Email address: 
Password: 
Password (again): 
Superuser created successfully.
```

`celery -A zlcelery worker -l info -B` 启动worker和beat

默认启动后celery不能多进程,所以当有其余进程任务是,不会运行.
比如我们用celery调用了ansible.ansible其实并不会执行,返回为空.
那么使用这个命令启动,即可.

`export PYTHONOPTIMIZE=1 && celery -A zlcelery worker -l info -B` 

动态增减 [参考](https://blog.csdn.net/vintage_1/article/details/47664297)

`export PYTHONOPTIMIZE=1 && celery -A zlcelery worker --autoreload -l info -B`


还有一种方法.未作真实性验证:

2.disable python packet multiprocessing process.py line 102: 
assert not _current_process._config.get(‘daemon’), \ ‘daemonic processes are not allowed to have children’
注释掉python包multiprocessing下面process.py中102行，关闭assert


然后我们访问到`ip:port/admin`登录即可看到相关的celery定时任务配置前端页面.


`tasks.py`是可以调用其他库的,比如ansible 

```
from zl.ansible_api.get_sys_info import GetSysInfoL

@shared_task
def get_date(ip,user,pwd):
    ip = ip
    user = user
    pwd = pwd
    get_res = GetSysInfoL(username=user,password=pwd,ip=ip)
    res=get_res.get_info()

    return res

# @shared_task 装饰器
# ip user pwd等信息是前端页面传来的 格式djcelery规定好的
# ["192.168.1.xx","root","xxxxxxx"]

```

##### 调用DB

相关的定制化,其实就是操作djcelery的数据库.不再从admin页面进入.
自己设计页面通过django-celery 的数据库CURD来设置任务计划

```
#导入先关的 模块

from djcelery.models import PeriodicTask, CrontabSchedule
from djcelery.schedulers import ModelEntry, DatabaseScheduler
from djcelery import loaders
```
