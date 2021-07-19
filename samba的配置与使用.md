## ubuntu 下 samba 服务端和客户端安装使用
### 服务端
#### 1、ubutnu 安装 samba 服务端
```
$ sudo apt-get install samba samba_common
```

#### 2、为用户创建 samba 密码
先添加用户码
```
# xxx 为系统用户名
$ sudo smbpassword -a xxx
```
然后执行命令会提示输入 samba 密码（注意提示是要 sudo 执行密码还是 samba 密码）

#### 3、创建共享目录
以在/opt目录下创建smbshare为例
```
$ sudo mkdir /opt/smbshare
$ sudo chmod 777 -R /opt/smbshare
```

#### 4、修改samba配置文件
samba 配置文件放在以下路径，用于控制samba目录和用户权限：
```
/etc/samba/smb.conf
```
修改配置前，建议备份配置文档
```
$ sudo cp /etc/samba/sam.conf /etc/samba/smbconf.bak
$ sudo vim /etc/samba/sam.conf
```
然后在 sam.conf 中添加如下配置:
```
[golbal]
# xxx 代表系统用户
security = xxx 
```
文件最后添加：
```
[share]
commnet = share folder
browseable = yes
path = /opt/smbshare
create mask = 0700
directory mask = 0700
# xxx 为系统用户名
valid users = xxx
force user = xxx
force group = xxx
public = yes
writable = yes
```
#### 5、samba 服务端的开启和关闭
```
$ sudo /etc/init.d/samba start
$ sudo /etc/init.d/samba stop
$ sudo /etc/init.d/samba restart
或者
$ sudo service samba start
$ sudo service samba stop
$ sudo service samba restart
```
有些情况下在``` /etc/init.d ```目录找不到samba这个可执行文件，但是能找到 smbd，这是可以用 smbd 代替
```
$ sudo /etc/init.d/smbd start
$ sudo /etc/init.d/smbd stop
$ sudo /etc/init.d/smbd restart
```

#### 6、samba 服务端开机启动
想要系统每次开机自动开启 samba server 服务，可以在 ```/etc/rc.local``` 文件中添加如下命令，没有rc.local,可以用创建一个后添加
```
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.


/etc/init.d/smbd start
# 或者 /etc/init.d/samba start 

exit 0

```

### 客户端

很多情况可能都是用 Windows 系统作为客户端的，当然如果有需求用 Ubuntu 访问 samba 服务，那么可以按照下面操作安装和使用。

#### 1、安装客户端
```
$ sudo apt-get install smbclient
```
#### 2、挂载文件
例如将服务端 ```/opt/share``` 文件夹挂载到客户端的 ```/opt``` 目录下
```
# xxx 为 samba 服务端用户名，yyy 为密码
$ sudo mount -t cifs //192.168.1.5/opt/share /opt -o username=xxx,password=yyy
```
#### 3、客户端卸载已挂载的目录
```
$ sudo umount /opt
```



