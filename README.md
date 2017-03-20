# zabbix-wechat

------

作者:superbigsea 邮箱:superbigsea@sina.com qq群238705984

做出来的效果 http://mp.weixin.qq.com/s?__biz=MzIzNjUxMzk2NQ==&mid=2247484693&idx=1&sn=ae7482230a90476f912eaada2849c861&chksm=e8d7fad7dfa073c17b364e15fe4464d2b1f09a49fcc66906856e72ff15cc2b7fc183af7f225a&mpshare=1&scene=23&srcid=0228sGfPKDoLskzjzaMv5j1b#rd

作为目前全球最大的男性同性交友平台，还是插一段中文的交友信息啊。
about me:前解放军军官，现在从事云计算工作。为了儿时理想，加入军队服役10年，参加过多次重要任务；为了青年时期理想现在从事云计算行业（青年时期买不起geforce 6800，于是诞生了能不能租用别人的显卡通过网络传输画面方式打游戏）。谈完理想，切入交友正题，公司在北京和上海都招人我们只有15薪、八险一金、餐补、通讯补、交通补、电脑补这些我都不想说了，每年也就两次调薪而已，每天下午茶、每月生日会。拉钩网页https://www.lagou.com/gongsi/j149630.html
想来和我把监控报警做的更加极致欢迎来打扰。
------
# Introduction

The purpose of this project is to build up an  auto-notification  alarm system which is compatible with most monitoring system,which includes the functions such as reminder alarm, alarm compression, alarm classification, alarm summary report etc... Readers need to have some  experiences in linux, python, zabbix.  Requires python3 support.

## ScreenShots
![](https://github.com/superbigsea/zabbix-wechat/blob/master/s1.png)
![](https://github.com/superbigsea/zabbix-wechat/blob/master/s2.png)
![](https://github.com/superbigsea/zabbix-wechat/blob/master/s3.png)
![](https://github.com/superbigsea/zabbix-wechat/blob/master/s4.png)
![](https://github.com/superbigsea/zabbix-wechat/blob/master/s5.png)
![](https://github.com/superbigsea/zabbix-wechat/blob/master/s6.png)
# Flow
![](https://github.com/superbigsea/zabbix-wechat/blob/master/4.PNG)

# Required components

## 1.An zabbix server alert scripts: Its function is to put the alarm information generated by zabbix server in a certain format.This scripts requires python3 support
## 2.Other components：mysql、redis(Used to cache wechat's token, you can also use memcacahed),tt server(Object storage,you can also use others like swift or ceph.)
## 3.A secondary domain with 80 and 1978 ports opened.
## 5.A WeChat Official Accounts （微信企业号）. It can be registered in https://admin.wechat.com/ or https://qy.weixin.qq.com/  
# Install and deployt
 
The entire installation configuration is rather complex,needing plenty of  components.
## 1 Zabbix server alert scripts
### (1) Install python3.4 and other components
``` shell
 yum install python34 python34-pip python34-devel
pip3 install  pycrypto
```
### (2) Generate an rsa key pair
``` shell
ssh-keygen -b 4096 -t rsa -f /etc/zabbix/pub
mv /etc/zabbix/pub.pub /etc/zabbix/pub.key
```
### (3) Move zabbix_alarm_script/all.py to zabbix alertscripts dir and edit it 
``` shell
zabbix_server_charturl = "http://127.0.0.1/zabbix/chart.php"  ##Zabbix server  chart api
ttserver = "http://www.example.com:1978"   ##The url for ttserver over internet.
server_url = "http://www.example.com/getvalue" ##The url for main process  over internet.
```
### (4) Generate cookies
``` shell
curl -c /tmp/zabbix_cookie -d "name=Admin&password=zabbix&autologin=1&enter=Sign+in"  http://127.0.0.1/zabbix/index.php
```
Change  the username and passwd and zabbix url to your own setting.
### (5)Config zabbix action
``` shell
{TRIGGER.NAME}@@@{TRIGGER.DESCRIPTION}@@@{HOSTNAME}@@@{TRIGGER.SEVERITY}@@@{ITEM.ID}@@@{TRIGGER.STATUS}@@@{HOST.CONN}@@@{TRIGGER.HOSTGROUP.NAME}@@@{EVENT.ID}@@@
```
![](https://github.com/superbigsea/zabbix-wechat/blob/master/1.PNG)
![](https://github.com/superbigsea/zabbix-wechat/blob/master/2.PNG)


## 2 MySQL Server 
``` shell
yum install mariadb-server -y
systemctl enable mariadb
systemctl start mariadb
CREATE DATABASE `alarm` DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci; #
grant all privileges on *.* to alarm@localhost identified by 'alarm';
```
## 3 Redis server 
``` shell
yum install redis
systemctl enable redis
systemctl start redis
```
## 4 Ttserver  
``` shell
yum install bzip2-devel gcc zlib-devel -y
wget http://fallabs.com/tokyotyrant/tokyotyrant-1.1.41.tar.gz
wget http://fallabs.com/tokyocabinet/tokyocabinet-1.4.48.tar.gz
tar zxvf tokyocabinet-1.4.48.tar.gz
cd tokyocabinet-1.4.48
./configure
make && make install 
 tar zxvf tokyotyrant-1.1.41.tar.gz
 cd tokyotyrant-1.1.41
 ./configure
make && make install 
mkdir /ttdata/
./ttserver -port 1978 -thnum 8 -tout 30 -dmn -pid /ttdata/tt.pid -kl -log /ttdata/tt.log -le -ulog /ttdata -ulim 128m -sid 1 -rts /ttdata/tt.rts /ttdata/ttdb.tch

```
## 5 Python3+django and other configurations
``` shell
 yum install python34 python34-pip python34-devel
pip3 install django  pymysql django_crontab redis  pycrypto
```
###  (1) Move the rsa private key which made in above  to  /etc/zabbix/pri.key
###  (2) Edit /etc/zabbix/wechat.conf
``` shell
[wechat]
corp_id=******************
corp_secret=*******************
toparty=***********
agentid=*************
url=http://**************
[redis]
host=****
port=****
```

The top 4 configuration can be referenced  in  https://github.com/X-Mars/Zabbix-Alert-WeChat.

url:The url for main process  over internet
### (3) Database configuration
```
vim zabbixwechat/settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql', # Add 'postgresql_psycopg2', 'mysql', 'sqlite3' or 'oracle'.
        'NAME': 'alarm',                      # Or path to database file if using sqlite3.
        # The following settings are not used with sqlite3:
        'USER': 'alarm',
        'PASSWORD': 'alarm',
        'HOST': 'alarm',                      # Empty for localhost through domain sockets or '127.0.0.1' for localhost through TCP.
        'PORT': '3306',                      # Set to empty string for default.
        'TEST_CHARSET': 'utf8',
        'TEST_COLLATION': 'utf8_general_ci',
    }
}

``` 
### (4) Initialize the database
python3  manage.py  makemigrations
### (5) WeChat Official Accounts Configuration

The configuration of the permissions of the message，also  reference https://github.com/X-Mars/Zabbix-Alert-WeChat
If you want to add the Summary Table
 on the menu of Wechat：
1.add the trusted domain name
 ![](https://github.com/superbigsea/zabbix-wechat/blob/master/3.PNG)
2 . add an jump to

https://open.weixin.qq.com/connect/oauth2/authorize?appid=wx7dec596b2599614c&redirect_uri=http%3a%2f%2f*********%2fall&response_type=code&scope=snsapi_base&state=1&connect_redirect=1#wechat_redirect
in the configuration。
Replace ******* with the urlencode domain name

http://tool.chinaz.com/tools/urlencode.aspx This URL can encode the domain name

### (6) Run 

python3  manage.py runserver 0.0.0.0:80

Enjoy!!!



# 一、简介

本项目目的是在微信端搭建一个兼容各个监控系统的统一报警处理系统，能实现报警提醒，报警压缩，报警分类，报警汇总报表等功能。主服务器实质上是一台位于公网上的http服务器，位于各地的zabbix server将报警信息经过初步处理后通过http post请求将信息传送至主服务器，主服务器将信息经过一定的处理再完成和微信端（报警接受者）进行处理，上文中的功能都可以实现，需要使用者有一定的linux、python、zabbix基础。
# 二 流程
![](https://github.com/superbigsea/zabbix-wechat/blob/master/%E6%8A%A5%E8%AD%A6%E6%B5%81%E7%A8%8B.png)

# 三 所需要的组件

## 1、zabbix server端报警脚本，主要将zabbix server传递过来的报警信息进行一定的格式处理，进行需要python3的支持
## 2、主服务器：采用python3+django
## 3、其他组件：mysql、redis（缓存微信token，也可以用memcacahed）、tt server（用来存放报警的图片和文字信息，不考虑高可用的话直接用Apache也可以）
## 4、一个开启了80和1978端口的二级域名 
## 5、一个微信企业号
# 四 安装部署 
 
 整个安装配置较为复杂，使用的组件较多
## (一)zabbix server 报警插件
### 1、生成密钥对  
``` shell
ssh-keygen -b 4096 -t rsa -f /etc/zabbix/pub
mv /etc/zabbix/pub.pub /etc/zabbix/pub.key
```
### 2、安装python3将zabbix_alarm_script/all.py拷贝到zabbix的alertscripts目录，编辑下面3个地方
``` shell
zabbix_server_charturl = "http://127.0.0.1/zabbix/chart.php"  ##zabbix server 绘图接口
ttserver = "http://www.example.com:1978"   ##公网的ttserver 安装步骤见下面
server_url = "http://www.example.com/getvalue" ##公网的主服务器接口
```
### 3、生成cookies
``` shell
curl -c /tmp/zabbix_cookie -d "name=Admin&password=zabbix&autologin=1&enter=Sign+in"  http://127.0.0.1/zabbix/index.php
```地址更改为zabbix server的地址，生成cookie的目的是便于脚本绘图时候不用认证了。
### 5、配置zabbix的action
``` shell
{TRIGGER.NAME}@@@{TRIGGER.DESCRIPTION}@@@{HOSTNAME}@@@{TRIGGER.SEVERITY}@@@{ITEM.ID}@@@{TRIGGER.STATUS}@@@{HOST.CONN}@@@{TRIGGER.HOSTGROUP.NAME}@@@{EVENT.ID}@@@
```
![](https://github.com/superbigsea/zabbix-wechat/blob/master/1.PNG)
![](https://github.com/superbigsea/zabbix-wechat/blob/master/2.PNG)
action 只需要传送一个参数，

## （二）mysql server 安装
``` shell
yum install mariadb-server -y
systemctl enable mariadb
systemctl start mariadb
CREATE DATABASE `alarm` DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci; #
grant all privileges on *.* to alarm@localhost identified by 'alarm';
```
## （三）redis server 安装
``` shell
yum install redis
systemctl enable redis
systemctl start redis
```
## （四）ttserver 安装 
``` shell
yum install bzip2-devel gcc zlib-devel -y
wget http://fallabs.com/tokyotyrant/tokyotyrant-1.1.41.tar.gz
wget http://fallabs.com/tokyocabinet/tokyocabinet-1.4.48.tar.gz
tar zxvf tokyocabinet-1.4.48.tar.gz
cd tokyocabinet-1.4.48
./configure
make && make install 
 tar zxvf tokyotyrant-1.1.41.tar.gz
 cd tokyotyrant-1.1.41
 ./configure
make && make install 
mkdir /ttdata/
./ttserver -port 1978 -thnum 8 -tout 30 -dmn -pid /ttdata/tt.pid -kl -log /ttdata/tt.log -le -ulog /ttdata -ulim 128m -sid 1 -rts /ttdata/tt.rts /ttdata/ttdb.tch

```
## （五）python3+django 安装 以及其他配置
``` shell
 yum install python34 python34-pip python34-devel
pip3 install django  pymysql django_crontab redis  pycrypto
```
### 1、将 上一节中生成的私钥文件 copy到 /etc/zabbix/pri.key
### 2 、编辑/etc/zabbix/wechat.conf
``` shell
[wechat]
corp_id=******************
corp_secret=*******************
toparty=***********
agentid=*************
url=http://**************
[redis]
host=****
port=****
```
其中上面4条为微信企业号的配置，可以参考https://github.com/X-Mars/Zabbix-Alert-WeChat
url 为服务器80端口的域名 
### 2、配置数据库连接
```
vim zabbixwechat/settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql', # Add 'postgresql_psycopg2', 'mysql', 'sqlite3' or 'oracle'.
        'NAME': 'alarm',                      # Or path to database file if using sqlite3.
        # The following settings are not used with sqlite3:
        'USER': 'alarm',
        'PASSWORD': 'alarm',
        'HOST': 'alarm',                      # Empty for localhost through domain sockets or '127.0.0.1' for localhost through TCP.
        'PORT': '3306',                      # Set to empty string for default.
        'TEST_CHARSET': 'utf8',
        'TEST_COLLATION': 'utf8_general_ci',
    }
}

``` 
### 3、初始化数据库
python3  manage.py  makemigrations
### 微信企业号菜单配置
### 4、运行程序

python3  manage.py runserver 0.0.0.0:80
