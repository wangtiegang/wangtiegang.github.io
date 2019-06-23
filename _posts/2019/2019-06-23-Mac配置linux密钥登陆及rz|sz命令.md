---
layout:     post
title:      "Mac配置linux密钥登陆及rz|sz命令"
subtitle:   " \"生成rsa密钥对，安装lrzsz\""
date:       2019-06-23 18:38:00
author:     "wangtiegang"
header-img: ""
catalog: true
tags:
    - linux
    - mac
---

公司的服务器都是使用密钥对进行登陆验证，这跟我以前使用用户名和密码登陆有点差别，并且在mac上指定密钥登陆比windows做起来复杂一些，以下记录相关的操作。

### mac生成rsa密钥对

* 使用ssh-keygen命令
  
  ```shell
  # 1.指定生成密钥对邮箱
  # 2.回车后按提示输入密钥对对保存路径和名称，比如/Users/wangtiegang/.ssh/wangtiegang
  # 3.回车后按提示输入密钥对应对密码，这个密码可以为空，不想设置对话就连续两次回车
  ssh-keygen -t rsa -C "wangtiegang@xxx.com"
  ```

提示成功生成后，可以在/Users/wangtiegang/.ssh/下看到两个文件，wangtiegang(私钥)和wangtiegang.pub(公钥)

### 在linux上配置用户对公钥

* 新建用户
  
  ```shell
  # 新建用户，会在home目录下生成用户目录，同时生成一个同用户名的用户组
  useradd wangtiegang

  # 必须修改用户密码，否则用户无法登陆
  passwd -u -f wangtiegang
  ```

* 在用户目录下新建.ssh文件夹及authorized_keys文件
  
  ```shell
  # 新建目录
  cd /home/wangtiegang
  mkdir .ssh

  # 新建文件
  cd .ssh
  vi authorized_keys

  # 必须修改.ssh文件夹权限为700，authorized_keys权限为600. 既不能大也不能小，否则无法登陆
  chmod 700 .ssh 
  chmod 600 authorized_keys 

  # 由于是使用root账号创建的文件，所以必须将权限授权给wangtiegang用户
  chown -R wangtiegang:wangtiegang .ssh
  ```

* 将公钥内容写到authorized_keys文件中
  
  ```shell
  # 将wangtiegang.pub中到内容复制到authorized_keys中保存
  ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCyo8AGKgb8QdSCeN6jfocPz/DW4Ov5otZ9MGK5SsROL3ABt8OEquqbIP7ErQKrmddeMZvbJvdeQjjQqvPCaX0BymZti7iZ4VkpLHTQ4vHDQUbVz2JNnd+aztlC/EnVRykbv/bvzkze/XQYrzln4SyLOW95WBA+m37NYpMZ+TvG5sCSi7vrjjfBRZhTf919l4pnc0s6qJGmTEHQhFeI5Jw2Ut94z5ZID0XwIviMLCjxj7VtO3mFfQyDXdVb2ztwa089Yn4xImQvmcir9XDGSyyCM5eo3Zkrdc7nK1KyEnHdut+k3edPPtUPZlo7omCCtVaIydUKV0snl7kq6GaNlU0p wangtiegang@xxx.com
  ```

### Mac使用ssh命令指定密钥登陆服务器

```shell
# 指定密钥登陆
ssh -i /Users/wangtiegang/.ssh/wangtiegang wangtiegang@10.xxx.xx.12 -p 8822
```

### Mac支持rz|sz命令

rz/sz命令是非要好用的文件上传下载命令，在Windows的ssh软件中都有比较好的支持，在Mac中想使用得做些配置才行。

* 确认服务器是否安装lrzsz
  
  ```shell
  # 可以使用rpm命令，安装了则会显示安装包得版本，否则显示为空
  rpm -qa|grep lrzsz

  # 如果安装，可以查看lrzsz相关目录
  rpm -ql lrzsz
  
  # 如果没有安装，推荐使用yum安装
  yum -y install lrzsz
  ```

* Mac安装iTerm2
  
  Mac自带得命令终端不支持rz/sz，需要安装一个iTerm2，直接去 [官网](https://www.iterm2.com/) 下载安装即可

* 安装Homebrew
  
  Homebrew是Mac下一款类似apt-get、yum的软件包管理工具

  ```shell
  ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
  ```

* 使用Homebrew在Mac上安装lrzsz
  
  ```shell
  brew install lrzsz
  ```

* 配置iTerm2属性
  
  上https://github.com/mmastrac/iterm2-zmodem  下载项目中的两个sh文件

  ![lrzsz-1](/img/in-post/2019-06/lrzsz-1.png)

  将文件拷贝到/usr/local/bin文件夹中

  ![lrzsz-2](/img/in-post/2019-06/lrzsz-2.png)

  打开iTerm2的preferences，设置Triggers，如果Linux服务器上的lrzsz也已经装好，就可以直接使用rz，sz了

  ![lrzsz-3](/img/in-post/2019-06/lrzsz-3.png)

  ![lrzsz-4](/img/in-post/2019-06/lrzsz-4.png)
