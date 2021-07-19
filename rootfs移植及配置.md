# busybox下制作rootfs

## 1 根文件系统构建


## 2 根文件系统添加主机名和提示符以及登录密码
### 2.1 在 busybox menuconfig 中勾选上以下2项
```
Busybox Settings ------>

        Busybox Library Tuning ----->

                    [*] Username completion

                    [*] Fancy shell prompts
```

### 2.2 在跟文件系统的 ```/etc``` 目录下新建 hostname 文件，并添加自定义的主机名，例如：
```
riotboard
```

### 2.3 在 ```etc/init.d``` 目录下的rcS文件中添加如下内容
```
/bin/hostname -F /etc/hostname
```

### 2.4 在根文件 ```/etc``` 目录下，新建 profile 文件，并在该文件中添加如下内容
```
HOSTNAME =`/bin/hostname`

PS1=`[\u@\h:\W]\$ `

export HOSTNAME PS1
```
bash 有两级命令提示符，我们这里说的是第一级，这一级缺省的提示符是字符 “$” （超级用户是 “#” ）,我们可以通过修改 PS1 来自定义提示符，格式为：
```
PS1='command list'
```
command list参数如下：
```
\! 显示该命令的历史记录编号。
\# 显示当前命令的命令编号。 
\$ 显示$符作为提示符，如果用户是root的话，则显示#号。 
\\ 显示反斜杠。 
\d 显示当前日期。 
\h 显示主机名。 
\n 打印新行。 
\nnn 显示nnn的八进制值。 
\s 显示当前运行的shell的名字。 
\t 显示当前时间。 
\u 显示当前用户的用户名。 
\W 显示当前工作目录的名字。 
\w 显示当前工作目录的路径
```


### 2.5 登录密码设置
#### 2.5.1 在 ```/etc``` 目录下，新建 passwd 文件
根据是否需要登录密码在 passwd 文件中添加如下内容
需要登录密码:
```
root:x:0:0:root:/root:/bin/sh
```
不需要登录密码：
```
root::0:0:root:/home/root:/bin/sh
```
#### 2.5.2 修改 ```/etc/inittab```
注释掉 inittab 中的 
```
::respawn:-/bin/sh
```
添加 
```
::respawn:/sbin/getty -L ttyS0 115200 vt100
```
其中的 ttyS0 为调试打印的串口设备，不通平台系统，命名不通，在进入系统后可通过
```ls /dev ``` 命令来查找。

#### 2.5.3 修改密码
在串口终端输入 ```passwd```,系统会提示给 root 账户设置一个密码。输入密码并确认密码后，reboot 重启设备，此时登录时就需要用户名和密码了


#### 注:
<font color=red>
2.5.3 这步操作后会在/etc 目录下生成一个新的 passwd 文件，这样貌似5.1可以不用操作，待验证。
</font>

#### 2.6 改变 profile 和 passwd 的文件属性
```
chmod 775 etc/profile  etc/passwd
```

## 3 设置开机自动脚本
kernel 启动（已经被载入内存，开始运行，并已初始化所有的设备驱动程序和数据结构等）之后，由进程 init 进程 （1 号进程，可参阅[Linux下1号进程的前世今](https://blog.csdn.net/u013165704/article/details/80519339)）来运行根目录下的 linuxrc，其实际是一个指向的 ```/bin/busybox``` 的链接，也就是说系统起来后运行的第一个程序是 busybox 本身。busybox 首先解析 ```/etc/inittab``` 中的配置(inittab参数配置见上方根文件系统配置)，内容如下：
```
# Boot-time system configuration/initialization script.
::sysinit:/etc/init.d/rcS #rcS是一个进行初始化配置的脚本，首先被执行
 
# Example of how to put a getty on a serial line (for a terminal)
::respawn:/sbin/getty -L ttyS000 115200 vt100 -n root -I "Auto login as root ..."   （设置控制台属性）
# Stuff to do when restarting the init process
::restart:/sbin/init
# Stuff to do before rebooting
::ctrlaltdel:/sbin/reboot
::shutdown:/bin/umount -a -r

```
从配置中可见 rcS 是第一个进行初始化额配置的脚本，然后是设置控制台、crtl+alt+del 操作以及关机等操作时需运行的程序。所以我们可以在rcS脚本中来定义我们要开机运行的脚本。先看下 ```/etc/init.d/rcS``` 中的配置
```bash
#!/bin/sh

# Start all init scripts in /etc/init.d
# executing them in numerical order.
for i in /etc/init.d/S??* ;do

     # Ignore dangling symlinks (if any).
     [ ! -f "$i" ] && continue

     case "$i" in
	*.sh)
	    # Source shell script for speed.
	    (
		trap - INT QUIT TSTP #忽略HUP, INT, QUIT, TSTP信号，防止执行脚本被中断
		set start
		. $i
	    )
	    ;;
	*)
	    # No sh extension, so fork subprocess.
	    $i start
	    ;;
    esac
done
```
rcS 脚本先遍历 ```/etc/init.d/``` 目录中所有“ S +数字”开头的脚本（忽略其他链接文件），从数字最小的开始先执行，所以我们按照此规则将需要开机执行的脚本添加到 ```/etc/init.d``` 目录下。
以 ZLG M6G2C 开发板例，其中的用户脚本放在最后执行的 S90start_userapp.sh 中，如下图所示，靠前先执行的一些为系统配置脚本。

https://blog.csdn.net/Ivan804638781/article/details/80607756

![开机脚本](/assets/开机脚本.jpg)


## 几种根文件镜像制作工具及使用方法
### 1. mkcramfs 
### 2. mkyaffs 
### 3. mkjffs