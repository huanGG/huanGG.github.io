---
title: samba的安装和配置
tags:
  - 开发工具
translate_title: installation-and-configuration-of-samba
date: 2019-05-07 23:30:37
---
# 一、在线安装:
```
yum install samba
```

接下进行配置，假设用户名为changhuan，要共享的目录名为 /home/changhuan， 执行 vi /etc/samba/smb.conf 编辑配置文件: 
<!--more-->
```
; 在[global]下面加入以下代码
unix charset = UTF-8
dos charset = CP932


; 在 Share Definitions下添加以下代码

; 共享名
[changhuan]  
; comment是对该共享的描述，可以是任意字符串。
comment = work Directories

; 共享目录路径
path=/home/map

; browseable用来指定该共享是否可以浏览。
browseable = yes

; writable用来指定该共享路径是否可写。
writable = yes

; 允许访问该共享的用户
valid users= map

; public用来指定该共享是否允许guest账户访问。
; public = yes/no
```

设置连接密码，
```
smbpasswd -a changhuan 
```

启动smb服务并设置开机自启
```
service smb restart
chkconfig smb on
```
访问samba
在Mac系统中：Finder --> 前往 --> 连接服务器，在服务器地址一栏输入smb://hostname/changhuan（hostname是开发机的域名或ip地址），然后输入上面配置的用户名和密码即可。

# 二、离线安装
多数机器配置好 yum 源后可以直接命令行安装samba，但我碰到一台开发机无法访问公司的yum源，所以尝试离线安装。

首先下载 samba 的[源码](https://src.fedoraproject.org/repo/pkgs/samba/samba-3.2.5.tar.gz/0f7539e09803ae60a2912e70adf1c747/samba-3.2.5.tar.gz)，因为这台机器还是centos6，所以我下的版本比较老，读者可以自行选择合适的版本。

然后进入源码所在目录，执行如下命令进行安装
```
tar zxvf  samba-3.2.5.tar.gz
cd samba-3.2.5/source    
. /configure      
make
make install
```
samba 默认被安装到 /usr/local/samba 文件夹下，这时候还不能直接启动服务，
首先要将路径 /usr/local/samba/lib 加入到/etc/ld.so.conf， 然后执行 ldconfig 更新库文件。
然后将  samba-3.2.5/example/smb.conf.deufault 文件复制到 /usr/local/samba/lib下,并重命名为smb.conf，然后按照上面的配置进行修改。

最后执行下面的命令启动 smb 服务。
```
sudo /usr/local/samba/sbin/smbd -D
sudo /usr/local/samba/sbin/nmbd -D 
```


# 三、端口被限制的解决方法
有的开发机能ping通，但是smb连接不上，这是因为端口做了限制，smb默认使用的端口为UDP137、UDP138、TCP139、TCP445。而公司开发机开放端口限制为8000～9000，所以需要修改smb的默认端口。

具体操作为修改 smb.conf
在 [global] 段添加以下代码：
smb ports = 8732（8000～9000之间的一个数）

修改后重启smb，然而mac端还是无法访问smb服务，考虑修改了服务端的端口，而客户端是否依然是访问的原来的默认端口，导致smb连接失败？

顺着这个思路，将访问地址由 smb://hostname/changhuan  改为 smb://hostname:8732/changhuan, 随即访问成功！