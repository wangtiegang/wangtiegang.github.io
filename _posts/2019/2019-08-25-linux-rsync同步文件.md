---
layout:     post
title:      "Linux使用rsync同步备份文件"
subtitle:   ""
date:       2019-08-25 16:05:00
author:     "wangtiegang"
header-img: ""
catalog: false
tags:
    - linux
---

日常工作中备份是常见的事情，重要的包括数据库备份和文件备份，有些应用会在开发中就走了主从备份方案，在使用的过程中就是实时备份，有些小一点的单机应用，备份方案就没这么完善了，经常需要定时备份成文件，然后发生故障时通过文件手动恢复。比如oracle数据库可以使用rman做增量备份，但是有个问题，rman备份出来的文件跟数据库在同一台服务器，如果不把备份文件再移动到其他服务器上去，则还是存在服务器存储故障，丢失备份无法恢复的风险。

因此我们需要在备份完成之后，将数据备份文件复制到其他服务器。将文件复制到其他服务器，可能第一反应是使用scp命令，在备份脚本结束后调用scp将文件发送过去就好了，但是实际上使用scp的时候会发现有几个缺点不好解决

* 数据库备份文件存在时效性，过期的文件需要删除，scp只能复制，还需要在其他服务器上写定时删除脚本
* scp没办法增量备份，每次都是整个文件夹/文件备份过去，如果文件很大，耗时无法接受

有没有比scp更好的方法呢？当然有，那就是使用rsync命令进行传输。

> rsync（remote synchronize）是一个远程数据同步工具，可通过LAN/WAN快速同步多台主机间的文件，也可以使用rsync同步本地硬盘中的不同目录。rsync是用于取代rcp的一个工具，rsync使用所谓的 “rsync算法”来使本地和远程两个主机之间的文件达到同步，这个算法只传送两个文件的不同部分，而不是每次都整份传送，因此速度相当快

rsync的特点可以解决scp的问题

* 支持文件夹镜像同步，也就是说保持一模一样，如果源文件夹中删除了一个文件，备份机器也会同步删除这个文件，这样就不用单独写脚本去删除过期文件了
* 使用算法对比，只传输变化的部分，增量更新，所以速度非常快

### rsync安装

安装过程很简单，首先使用 ```rpm -qa|grep rsync``` 检查服务器上是否安装了rsync，如果没有安装则使用 ```yum``` 在线安装，或者下载 rpm 包离线安装。

### rsync服务端配置

> rsync需要把备份服务器和源文件服务器中的一台设为服务端，这样客户端服务器连接的时候，服务端可以进行权限验证

> 在服务器找到rsync的配置文件，一般用root用户在/etc/rsyncd.conf进行配置

```shell
use chroot = no
max connections = 4
strict modes =yes
port = 2049

#指定日志文件和日志格式
log file = /data2/backup/dms_rsync.log
transfer logging = yes
log format = %t %a %m %f %b

#配置module，传输服务端必须配置
[dmsbak]
path = /data2/backup/
comment = dms prod bak
ignore errors
read only = no
list = no
#可以使用此module的用户
#此用户不是操作系统用户，该用户可以在指定的secrets file中配置
auth users = dms_rsync
#指定secrets file.pas文件中配置可以可以访问的用户
secrets file = /etc/dms_rsync.pas
hosts allow = 10.xxx.xx.xx
hosts deny = 0.0.0.0/0

[cmsbak]
path = /data2/backup/
comment = cms prod bak
ignore errors
read only = no
list = no
#可以使用此module的用户
#此用户不是操作系统用户，该用户可以在指定的secrets file中配置
auth users = cms_rsync
#指定secrets file.pas文件中配置可以可以访问的用户
secrets file = /etc/cms_rsync.pas
hosts allow = 10.xxx.xx.xx
hosts deny = 0.0.0.0/0

```

> 创建secrets file：/etc/dms_rsync.pas，用来验证客户端同步的用户，该文件格式必须为用户名：密码

```shell
dms_rsync:testpass
```

> secrets file的权限必须为600

```shell
chmod 600 /etc/dms_rsync.pas
```

### 客户端配置


> 新建pas文件，保存访问用户的密码，在/home/dms下创建dms_rsync.pas文件，只输入保存密码

```
testpass
```
> 设置文件权限为600

```
chmod 600 /home/dms/dms_rsync.pas
```

> 运行测试命令

```shell
rsync -auzv --progress  --port=2049 --password-file=/home/dms/dms_rsync.pas /home/dms dms_rsync@10.xxx.xx.130::dmsbak/dms_prod_attachbak
# 详细参数可查文档，其中--password是客户端存储密码的文件 
# /home/dms是客户端需要传送的目录，也可以指定具体文件
# dms_rsync是服务端配置文件中指定可以访问dmsbak的用户
# dmsbak是服务端配置的module，dms_prod_attachbak是dmsbak配置的path下的文件夹，表示把文件传送到$path/dms_prod_attachbak下

```

如果配置没有问题，则可以看到文件已经同步了。

### 配置cron任务执行同步

在配置好rsync之后，我们只要在crontab中配置好定时任务，就可以定时传送备份文件到其他服务器，这样整个备份方案就比较可靠了。

> 查看cron是否按照和服务状态

```shell
rpm -qa|grep rsync
service crond status
```

> 添加crontab

```shell
#添加方式一
crontab -e #此方式是直接添加的当前用户的cron计划，缺点是每个用户的计划都在不同地方，无法统一查看

#添加方式二，切换到root用户
vi /etc/crontab
#在文件中添加计划，此处添加计划必须指定执行用户，所有计划集中管理，例添加dms用户的定时计划：
#[cron表达式] [用户] [命令]
*/3 * * * * dms rsync -auzv --progress  --port=2049 --password-file=/home/dms/dms_rsync.pas /data/hdm/dms_attachment dms_rsync@10.xxx.xxx.xxx::dmsbak/dms_prod_attachbak
```

> 执行日志在/var/logs/cron，部分机器找不到cron日志文件，应该是安装配置问题，可在root用户邮件中查看

```shell
tail -f /var/spool/mail/root
```
