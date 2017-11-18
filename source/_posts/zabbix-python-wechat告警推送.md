---
title: zabbix-python-wechat告警推送
date: 2017-03-31 12:00:24
tags: zabbix
categories: zabbix
copyright: true
---
<img src="/images/zabbix-text.jpg" alt="zbx-text" align=center />
**前言**
zabbx告警结合微信,可以更及时对故障响应,处理.
<!--more-->
___
**告警的模式分为两种,第一种就是本文消息的直接推送,第二种是图文推送,可做到交互,本文主要介绍第一种**

#### 前提条件
>1. zabbix
>2. 企业号
>3. 脚本

#### 企业号
##### 申请
不赘述.
##### 获取信息
取得企业号的部门ID,应用ID, CorpID,Secret

!['企业号信息'](/images/zabbix-2.png)


#### 脚本
脚本可以推送两种格式,文本和新闻.
##### 推送文本
**来自网上, 视情况修改4处 `self.__corpid` 和`self.__secret` `toparty` `agentid`**
```
#!/usr/bin/env python
# coding: utf-8

import urllib,urllib2,json
import sys
reload(sys)
sys.setdefaultencoding( "utf-8" )

class WeChat(object):
    __token_id = ''
    # init attribute
    def __init__(self,url):
        self.__url = url.rstrip('/')
        #CorpID
        self.__corpid = 'wx643f**********'   
        #Secret 
        self.__secret = 'YQOu7Qzad********hGJ9xHlby****_v1oF2WpBOlsy****TivMKAL***voA3MwKH'

    # Get TokenID
    def authID(self):
        params = {'corpid':self.__corpid, 'corpsecret':self.__secret}
        data = urllib.urlencode(params)

        content = self.getToken(data)

        try:
            self.__token_id = content['access_token']
            # print content['access_token']
        except KeyError:
            raise KeyError

    # Establish a connection
    def getToken(self,data,url_prefix='/'):
        url = self.__url + url_prefix + 'gettoken?'
        try:
            response = urllib2.Request(url + data)
        except KeyError:
            raise KeyError
        result = urllib2.urlopen(response)
        content = json.loads(result.read())
        return content

    # Get sendmessage url
    def postData(self,data,url_prefix='/'):
        url = self.__url + url_prefix + 'message/send?access_token=%s' % self.__token_id
        request = urllib2.Request(url,data)
        try:
            result = urllib2.urlopen(request)
        except urllib2.HTTPError as e:
            if hasattr(e,'reason'):
                print 'reason',e.reason
            elif hasattr(e,'code'):
                print 'code',e.code
            return 0
        else:
            content = json.loads(result.read())
            result.close()
        return content

    # send message
    def sendMessage(self,touser,message):

        self.authID()

# 具体可查看wechat接口文档http://qydev.weixin.qq.com/
        data = json.dumps({
            #'touser':touser, #企业号中的用户帐号,如果没,则按部门推送.
            'toparty':"3",  #部门ID 
            'msgtype':"text",
            'agentid':"1", #应用ID
            'text':{
                'content':message
            },
            'safe':"0"
        },ensure_ascii=False)
        response = self.postData(data)
        print response
if __name__ == '__main__':
    a = WeChat('https://qyapi.weixin.qq.com/cgi-bin')
    a.sendMessage(sys.argv[1], sys.argv[3])
```

##### 推送news
**同样的脚本,需要修改如下**
```
##图文格式(推荐),修改脚本的data如下
        data = json.dumps({
            'touser':touser, #企业号中的用户帐号
            'toparty':"3",  #部门ID 
            'msgtype':"news",
            'agentid':"1", #应用ID
            'news':{
                "articles":[{
                    "title":title,
                    "description":message
                        }]
                },   
            "safe":0
        },ensure_ascii=False)
##end
##修改
1：import re

2：
def sendMessage(self,touser,message): 修改为
def sendMessage(self,touser,title,message):

3：
if __name__ == '__main__':
    a = WeChat('https://qyapi.weixin.qq.com/cgi-bin')
    a.sendMessage(sys.argv[1], sys.argv[3])
修改为
if __name__ == '__main__':
    a = WeChat('https://qyapi.weixin.qq.com/cgi-bin')
    info = sys.argv[3]
    info2 = re.split('[&&]',info)
    a.sendMessage(sys.argv[1],info2[0],info2[2])

4：
zabbix页面的动作-默认信息中添加&& 用来进行信息分割
```
脚本放至zbx脚本目录下即可


#### zabbix

##### 媒介配置
>管理 ---> 媒介---> 新建
>名称随便,这里命名wechat
>脚本名称要和上面的一致wechat.py
>脚本参数要依次填写以下三个
>这三个参数就是传给脚本的sys.argv[]

```
{ALERT.SENDTO}
{ALERT.SUBJECT}
{ALERT.MESSAGE}
```

!['zabbix媒介'](/images/zabbix-3.jpg)

##### 用户
>创建一个wechat用户,给予相关用户媒介权限(刚刚创建的媒介)
>注意收件人,这个是脚本的第一个参数.也就是说这里要填写的是应该是微信用户，或者部门ID，这里填写部门ID 这样wechat.py脚本才能取得正确的值

!['zabbix用户'](/images/zabbix-4.jpg)

##### 动作
>动作的 操作选项-->选发送消息,添加相应的用户 或者群组就行了.
>我是发送给wechat用户的,因为他关联了wechat.py媒介.所以所有的消息将传递给他,他的媒介连接脚本,脚本连接企业微信,整个流程就打通了.

###### 参考配置

告警:
```
{TRIGGER.SEVERITY} : {TRIGGER.NAME}&&
-------------------------------------------
ID：{EVENT.ID}
主机名称:{HOST.NAME}
主机地址:{HOST.IP}
告警时间:{EVENT.DATE}  -  {EVENT.TIME}

告警描述:{TRIGGER.DESCRIPTION}
---------------------------------
```
恢复:
```
恢复 :  {TRIGGER.NAME}&&
-------------------------------------------
ID：{EVENT.ID}
主机名称:{HOST.NAME}
主机地址:{HOST.IP}
告警时间:{EVENT.DATE}  -  {EVENT.TIME}
恢复时间:{EVENT.RECOVERY.DATE}  -  {EVENT.RECOVERY.TIME}
持续时间:{EVENT.AGE}

告警等级: {TRIGGER.SEVERITY}
告警描述:{TRIGGER.DESCRIPTION}
```
!['zabbix动作'](/images/zabbix-5.jpg)



#### 图文(参考)
```
#!/usr/bin/env python3
# coding: utf-8
#authour lizili
import sys
import requests
from time import sleep
import json
import time

WX_CORPID = r"wx643f8***********"
WX_CORPSECRET = r"qwojyG*******************"
WX_TOKEN_FILE = "/tmp/accesstoken"
AGENT_ID = '100000*'
TOPARTY='*'
LOG_FILE="/tmp/zabbix_send_log"
def c_log(msg):
    try:
        perameter_file=open(LOG_FILE,'a')
        perameter_file.write(msg+"\n")
    except:
        exit(1)
    finally:
        perameter_file.close()
def refresh_token(corpid=WX_CORPID, corpsecrt=WX_CORPSECRET):
    get_url = r"https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=" + WX_CORPID + "&corpsecret=" + WX_CORPSECRET
    for i in range(3):
        try:
            req_ret = requests.get(get_url, timeout=6)
            jso_ret = req_ret.json()
            file_tok = open(WX_TOKEN_FILE, "w")
            file_tok.write(jso_ret["access_token"])
            file_tok.close()
            return jso_ret["access_token"]
        except requests.exceptions.ConnectTimeout:
            sleep(10)
        except Exception as exc:
            print(exc.args)
            return None
    return None

def get_token(corpid=WX_CORPID, corpsecrt=WX_CORPSECRET):
    try:
        file_tok = open(WX_TOKEN_FILE, "r")
        token = file_tok.read()
        file_tok.close()
        if len(token) !=64:
            raise FileNotFoundError(r"token length !=64")
        return token
    except FileNotFoundError:
        get_url = r"https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=" + WX_CORPID + "&corpsecret=" + WX_CORPSECRET
        for i in range(3):
            try:
              req_ret = requests.get(get_url, timeout=6)
              jso_ret = req_ret.json()
              file_tok = open(WX_TOKEN_FILE, "w")
              file_tok.write(jso_ret["access_token"])
              file_tok.close()
              return jso_ret["access_token"]
            except requests.exceptions.ConnectTimeout:
                sleep(10)
            except Exception as exc:
                print(exc.args)
                return None
    except Exception as exc:
            print(exc.args)
    return None

def create_articles():
    msg_argv=sys.argv[3].split("&&")
    if(msg_argv[0]=="PROBLEM"):
        if(msg_argv[1]=="灾难"):
            pic_url="图片地址"
        elif(msg_argv[1]=="一般严重"):
            pic_url="图片地址"
        elif(msg_argv[1]=="严重"):
            pic_url="图片地址"
        elif(msg_argv[1]=="警告"):
            pic_url="图片地址"
        elif(msg_argv[1]=="通知"):
            pic_url="图片地址"
        else:
            pic_url="图片地址"
        ret = [
            {
                "title": "%s"%(msg_argv[2]),
                "description":"ID:%s\n告警时间: %s"%(msg_argv[3],msg_argv[7]),
            },
            {
                "title":"主机分类: %s \n主机名称: %s \n主机地址: %s"%(msg_argv[4],msg_argv[5],msg_argv[6]),
                "picurl": "%s"%pic_url
            },
            {
                "title":"告警描述:\n%s"%(msg_argv[8]),
            }
        ]
        return ret
    elif(msg_argv[0]=="OK"):
        if(msg_argv[1]=="通知"):
            return None
        else:
            pic_url="图片地址"
            return [
                {
                "title": "恢复-%s"%(msg_argv[2]),
                "description":"ID: %s\n持续时长：%s\n恢复时间: %s\n告警时间: %s"%(msg_argv[3],msg_argv[10],msg_argv[7],msg_argv[9]),
                },
                {
                "title":"主机分类: %s\n主机名称: %s\n主机地址: %s"%(msg_argv[4],msg_argv[5],msg_argv[6]),
                "picurl": "%s"%pic_url
                },
                {
                "title":"告警等级: %s\n告警描述: %s"%(msg_argv[1],msg_argv[8])
                }
            ]
    return None

#def send_news(type,target,arti):
def send_news(arti):
    access_token = get_token()
    if access_token == None:
        return 1
    send_msg_url = r"https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token="+access_token
    msg_josn ={}
    msg_josn["toparty"] = TOPARTY
    msg_josn["msgtype"] = "news"
    msg_josn["agentid"] = AGENT_ID

    msg_josn["news"] = {"articles": arti}
    #print (msg_josn)
    send_data = json.dumps(msg_josn, ensure_ascii=False).encode(encoding='UTF8',)   
    #print(send_data)
    for i in range(4):
        try:
            if i >= 4:
                return 1
            req_ret = requests.post(send_msg_url, data=send_data, timeout=5)
            c_log(req_ret.text)
            jos_ret = req_ret.json()

            if jos_ret["errcode"] == 0:
                return 0
            elif jos_ret["errcode"] == 40014:
                token = refresh_token(WX_CORPID, WX_CORPSECRET)
                if token == None:
                    return 1
                access_token = token
                send_msg_url = r"https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=" + access_token
            else:
                return 1
        except requests.exceptions.ConnectTimeout:
            sleep(2)
        except Exception as exc:
            return 1
    return 1

if __name__=="__main__":
    articles=create_articles()
    #if(articles==None):
     #   exit(1)
    #send_news(0,sys.argv[1],articles)
#    send_news(articles)
    if(send_news(articles)!=0):
        exit(1)

#告警
{TRIGGER.STATUS}&&{TRIGGER.SEVERITY}&&{HOST.NAME}-{TRIGGER.NAME}&&{EVENT.ID}&&{TRIGGER.HOSTGROUP.NAME}&&{HOST.NAME}&&{HOST.IP}&&{EVENT.DATE}-{EVENT.TIME}&&{TRIGGER.DESCRIPTION}

#恢复
{TRIGGER.STATUS}&&{TRIGGER.SEVERITY}&&{HOST.NAME}-{TRIGGER.NAME}&&{EVENT.ID}&&{TRIGGER.HOSTGROUP.NAME}&&{HOST.NAME}&&{HOST.IP}&&{EVENT.DATE}  -  {EVENT.TIME}&&{TRIGGER.DESCRIPTION}&&{EVENT.RECOVERY.DATE}  -  {EVENT.RECOVERY.TIME}&&{EVENT.AGE}
```
可推送出类似如下格式的告警

<img src="/images/zabbix-6.jpg" width = "30%" height = "20%" align=center />

#### 图文(参考)进阶
> 对web开发有一定了解,熟悉python,zabbix API,微信API,用Django做后台处理交互.可推出如下告警.
> 有想法的可研究,过程复杂,三言两语讲不清.

<img src="/images/zabbix-7.jpg" width = "30%" height = "20%" align=center />

完
___
