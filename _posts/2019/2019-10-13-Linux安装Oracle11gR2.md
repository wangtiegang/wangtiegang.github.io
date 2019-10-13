---
layout:     post
title:      "Linux安装Oracle11gR2"
subtitle:   "CentOS7.6 Oracle11.2.0.4"
date:       2019-10-13 19:12:00
author:     "wangtiegang"
header-img: ""
catalog: true
tags:
    - Linux
    - Oracle
---

最近两天在Linux上折腾安装Oracle数据库，本来保险起见应该叫dba来安装的，但是最后这个事情落到了我头上，只能拿着一个相近的文档边安装边学习了，此文详细记录安装过程。

### 安装信息

* 操作系统：CentOS 7.6
* Oracle版本：11g 11.2.0.4

### 操作系统环境准备

#### /etc/sysctl.conf设置（root）

修改系统内核参数，在/etc/sysctl.conf中添加以下内容

```shell
fs.file-max = 6815744
kernel.shmmni = 4096
kernel.sem = 256 32000 100 142
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048586
```

#### 安装依赖包（root）

```shell
yum install -y gcc libaio glibc.i686 compat-libstdc++-33 compat-libstdc++-33.i686 elfutils-libelf-devel glibc-devel glibc-headers gcc-c++ libaio-devel libaio-devel.i686 libgcc.i686 libstdc++ libstdc++.i686 unixODBC unixODBC.i686 unixODBC-devel unixODBC-devel.i686 ksh
```

#### 用户组和用户创建（root）

```shell
groupadd oinstall
groupadd dba 
useradd -g oinstall -g dba -m oracle
passwd oracle
```

#### 设置/etc/security/limits.conf（root）

> 在文件尾追加如下内容，修改限制，增加oracle性能

```shell
@dba hard nofile 65536
@dba soft nofile 65536
@dba hard nproc 16384
@dba soft nproc 16384
@dba hard stack 16384
@dba soft stack 10240
```

#### 配置/etc/hosts（root）

> 设置host是为了安装oracle使用别名，方便以后切换ip

```shell
10.xxx.xx.78 erpuatapp.xx.xxx.com erpuatapp
```

#### 在/home/oracle下创建11g.env（oracle）

> 在11g.env中配置一些环境变量信息，然后加入oracle的.bash_profile文件，这样在登陆oracle用户时就会自动应用。

```shell
Vi 11g.env

# 文件添加如下内容
# oracle
ORACLE_BASE=/data/oracle
export ORACLE_BASE
ORACLE_HOME=$ORACLE_BASE/product/11.2.0/dbhome_1
export ORACLE_HOME
ORACLE_SID=FINTEST
export ORACLE_SID
PATH=$ORACLE_HOME/bin:$PATH
export PATH
LD_LIBRARY_PATH=$ORACLE_HOME/lib:/usr/X11R6/lib:/usr/openwin/lib:$ORACLE_HOME/lib:/usr/dt/lib:$ORACLE_HOME/ctx/lib
export LD_LIBRARY_PATH
NLS_LANG="American_America.AL32UTF8"
export NLS_LANG
NLS_DATE_FORMAT="DD-MON-RR"
export NLS_DATE_FORMAT
NLS_NUMERIC_CHARACTERS=".,"
export NLS_NUMERIC_CHARACTERS
NLS_SORT="binary"
export NLS_SORT
export PATH=$ORACLE_HOME/OPatch:$PATH
export TNS_ADMIN=$ORACLE_HOME/network/admin
```

编辑 .bash_profile文件，在末尾加入

```shell
. $HOME/11g.env
```

#### 设置oraInst.loc（root）

创建文件/etc/oraInst.loc，加入内容

```shell
inventory_loc=/data/oracle/oraInventory
inst_group=dba
```

以oracle用户创建inventory目录

```shell
mkdir -p /data/oracle/oraInventory
chmod 777  /data/oracle /oraInventory
```

#### 上传安装文件

安装介质
p13390677_112040_Linux-x86-64_1of7.zip
p13390677_112040_Linux-x86-64_2of7.zip

OPatch更新包
p6880880_112000_Linux-x86-64.zip

补丁包
p24006111_112040_Linux-x86-64.zip


### 配置vnc服务

安装数据库的方式有多种，图形化安装，静默安装，克隆安装。如果有图形化界面辅助就会简单很多，因此需要开启服务器vnc服务。

在这一步浪费了非常多的时间，本来安装vnc服务非常简单，但是这台服务器的 /usr 等系统目录只有2G大小，随便安装一些包就只剩200M空间了，在yum安装vnc的时候默认目录空间不足，无法安装。于是我只能指定安装目录，安装完成之后，将指定目录的/bin，/lib等目录配置到系统，启动vnc的时候出现了各种错误，一时无法解决，最后求助于dba，dba一开始建议重装系统，因为系统盘划分的太不合理了，但是服务器上已经有其他服务了，所以没有采用，最后采用了另外一种方法。

* 找一台安装了vnc的服务器A，使用vnc连接
* ssh到服务器B上，执行 ```export DISPLAY=ip:1.0```，ip为A服务器，1.0为A服务器开启的vnc桌面编号，将图形化界面传送到A服务器
* 在A服务器的命令窗口中执行 ```xhost + ``` 给其他机器授权
* 然后在B服务器执行安装命令，图形界面就可以传到A的vnc了

### Oracle安装

解压p13390677_112040_Linux-x86-64_1of7.zip，p13390677_112040_Linux-x86-64_2of7.zip

```shell
cd /home/oracle_soft/database
./runInstaller
```

从A的vnc界面操作

选择不接收更新
![oinstall-1](/img/in-post/2019-10/oinstall-1.png)
![oinstall-2](/img/in-post/2019-10/oinstall-2.png)
![oinstall-3](/img/in-post/2019-10/oinstall-3.png)
![oinstall-4](/img/in-post/2019-10/oinstall-4.png)
![oinstall-5](/img/in-post/2019-10/oinstall-5.png)
![oinstall-6](/img/in-post/2019-10/oinstall-6.png)
![oinstall-7](/img/in-post/2019-10/oinstall-7.png)

目录为oracle安装路径，跟11g.env中一致
![oinstall-8](/img/in-post/2019-10/oinstall-8.png)

选择安装组为dba组，OSOPER为空
![oinstall-9](/img/in-post/2019-10/oinstall-9.png)
![oinstall-10](/img/in-post/2019-10/oinstall-10.png)

忽略检查警告
![oinstall-11](/img/in-post/2019-10/oinstall-11.png)
![oinstall-12](/img/in-post/2019-10/oinstall-12.png)
![oinstall-13](/img/in-post/2019-10/oinstall-13.png)
![oinstall-14](/img/in-post/2019-10/oinstall-14.png)

安装过程中会有一个报错，先去服务器上修改配置，再点击retry

```
Exception String: Error in invoking target 'agent nmhs' of makefile '/data/oracle/product/11.2.0/dbhome_1/sysman/lib/ins_emagent.mk'
```

![oinstall-15](/img/in-post/2019-10/oinstall-15.png)

编辑/data/oracle/product/11.2.0/dbhome_1/sysman/lib/ ins_emagent.mk 文件，在$(MK_EMAGENT_NMECTL)后面增加 –lnnz11，然后retry，问题解决，继续安装。

![oinstall-16](/img/in-post/2019-10/oinstall-16.png)

此处需要使用root用户去执行提示中的脚本文件
![oinstall-17](/img/in-post/2019-10/oinstall-17.png)

默认[/usr/local/bin]不变，直接回车，然后按提示选择y或n直到完成
![oinstall-18](/img/in-post/2019-10/oinstall-18.png)

此时数据库安装完成
![oinstall-19](/img/in-post/2019-10/oinstall-19.png)

### 安装官方补丁

11g安装补丁前需要先更新opatch版本

备份OPatch目录

```mv $ORACLE_HOME/OPatch $ORACLE_HOME/OPatch.pre6880880```

将新的OPatch版本解压到安装目录

```unzip -d $ORACLE_HOME p6880880_112000_Linux-x86-64.zip```

验证版本是否更新
![oinstall-20](/img/in-post/2019-10/oinstall-20.png)

解压补丁包

```unzip p24006111_112040_Linux-x86-64.zip```

```cd 24006111/```

打补丁前检查是否存在补丁冲突：

```
export PATH=$PATH:$ORACLE_HOME/OPatch
opatch prereq CheckConflictAgainstOHWithDetail -ph ./
```

![oinstall-21](/img/in-post/2019-10/oinstall-21.png)

应用补丁

```opatch apply```

如果报错

```
Missing command :fuser
Prerequisite check "CheckSystemCommandAvailable" failed.
```

![oinstall-22](/img/in-post/2019-10/oinstall-22.png)

则需要使用root安装psmisc依赖，安装完成重新执行opatch apply

```yum install -y psmisc```

### 创建数据库

```
cd $ORACLE_HOME/bin
./dbca
```

![oinstall-23](/img/in-post/2019-10/oinstall-23.png)
![oinstall-24](/img/in-post/2019-10/oinstall-24.png)
![oinstall-25](/img/in-post/2019-10/oinstall-25.png)

输入数据库名和SID

![oinstall-26](/img/in-post/2019-10/oinstall-26.png)

![oinstall-27](/img/in-post/2019-10/oinstall-27.png)
![oinstall-28](/img/in-post/2019-10/oinstall-28.png)

设置密码

![oinstall-29](/img/in-post/2019-10/oinstall-29.png)
![oinstall-30](/img/in-post/2019-10/oinstall-30.png)

选择数据文件存储目录
![oinstall-31](/img/in-post/2019-10/oinstall-31.png)
![oinstall-32](/img/in-post/2019-10/oinstall-32.png)

![oinstall-33](/img/in-post/2019-10/oinstall-33.png)
![oinstall-34](/img/in-post/2019-10/oinstall-34.png)
![oinstall-35](/img/in-post/2019-10/oinstall-35.png)

这个大小可以根据服务器内存大小合理设置，一般加起来不超过总内存70%

![oinstall-36](/img/in-post/2019-10/oinstall-36.png)
![oinstall-37](/img/in-post/2019-10/oinstall-37.png)


![oinstall-38](/img/in-post/2019-10/oinstall-38.png)
![oinstall-39](/img/in-post/2019-10/oinstall-39.png)
![oinstall-40](/img/in-post/2019-10/oinstall-40.png)

将控制文件放在同一路径下：

![oinstall-41](/img/in-post/2019-10/oinstall-41.png)

修改所有的redo log的File Size为200M

![oinstall-42](/img/in-post/2019-10/oinstall-42.png)
![oinstall-43](/img/in-post/2019-10/oinstall-43.png)
![oinstall-44](/img/in-post/2019-10/oinstall-44.png)
![oinstall-45](/img/in-post/2019-10/oinstall-45.png)
![oinstall-46](/img/in-post/2019-10/oinstall-46.png)

至此数据库创建完成。

### 数据库监听设置

#### 创建listener和配置TNS

```
cd $ORACLE_HOME/bin
./ netca
```

![oinstall-47](/img/in-post/2019-10/oinstall-47.png)

![oinstall-48](/img/in-post/2019-10/oinstall-48.png)
![oinstall-49](/img/in-post/2019-10/oinstall-49.png)
![oinstall-50](/img/in-post/2019-10/oinstall-50.png)
![oinstall-51](/img/in-post/2019-10/oinstall-51.png)
![oinstall-52](/img/in-post/2019-10/oinstall-52.png)
![oinstall-53](/img/in-post/2019-10/oinstall-53.png)
![oinstall-54](/img/in-post/2019-10/oinstall-54.png)
![oinstall-55](/img/in-post/2019-10/oinstall-55.png)
![oinstall-56](/img/in-post/2019-10/oinstall-56.png)
![oinstall-57](/img/in-post/2019-10/oinstall-57.png)

此处配置hosts中的名称

![oinstall-58](/img/in-post/2019-10/oinstall-58.png)
![oinstall-59](/img/in-post/2019-10/oinstall-59.png)
![oinstall-60](/img/in-post/2019-10/oinstall-60.png)
![oinstall-61](/img/in-post/2019-10/oinstall-61.png)
![oinstall-62](/img/in-post/2019-10/oinstall-62.png)
![oinstall-63](/img/in-post/2019-10/oinstall-63.png)

#### 设置local_listener

```
sqlplus /nolog
conn / as sysdba
Alter system set local_listener=FINTEST scope=both;
alter system register;
```

重启数据库

```shell
#停止：
lsnrctl stop FINTEST
sqlplus /nolog
conn / as sysdba
shutdown immediate

#启动
lsnrctl start FINTEST
sqlplus /nolog
conn / as sysdba
startup
```

使用plsql连接数据库，测试报错

```
ORA-12514: TNS:listener does not currently know of service requested in connect
Descriptor
```

网上查询之后，修改文件/data/oracle/product/11.2.0/dbhome_1/network/admin/ listener.ora，增加SID_LIST_FINTEST

![oinstall-64](/img/in-post/2019-10/oinstall-64.png)

然后重启监听问题解决，通常先启动监听，再启动数据库，这个SID_LIST_FINTEST是会动态注册的，但是不知道为什么没有注册成功，所以直接修改文件静态注册。

至此数据库安装，创建数据库全部完成。