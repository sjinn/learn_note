1 uboot移植
https://blog.csdn.net/qq_16777851/article/details/81543373
1）注意xxx_board下kconfig的内容

config SYS_BOARD --- 板子名称，即xxx_board这个文件夹名

config SYS_VENDOR --- 芯片或板子所属厂商，即xxx_board文件夹上一级文件名

config SYS_CONFIG_NAME  --- include/configs下所以对xxx_board 头文件名称，一般命名为xxx_board.h

2）新板子在xxx_board/xxx_board.cfg 或者头文件夹下plugin.S中添加 DRAM的校准参数（imx系，其他不确定，还未接触过）

3）如果uboot中用到了DTS，需要在/arch/arm/dts makefile 中在对应架构芯片下添加dts,xxx_board文件夹下的xxx_board_defconfig中需要添加CONFIG_DEFAULT_DEVICE_TREE=对应的dts文件名(不带末尾格式)

4）


2、uboot.imx与uboot.bin的关系，以及uboot.imx解析
https://blog.csdn.net/zhaoyun_zzz/article/details/84990606

3、mmc 命令
```
版本：U-Boot 2017.03

mmc info - display info of the current MMC device
mmc read addr blk# cnt
mmc write addr blk# cnt
mmc erase blk# cnt
mmc rescan
mmc part - lists available partition on current mmc device
mmc dev [dev] [part] - show or set current mmc device [partition]
mmc list - lists available devices
mmc hwpartition [args...] - does hardware partitioning
  arguments (sizes in 512-byte blocks):
    [user [enh start cnt] [wrrel {on|off}]] - sets user data area attributes
    [gp1|gp2|gp3|gp4 cnt [enh] [wrrel {on|off}]] - general purpose partition
    [check|set|complete] - mode, complete set partitioning completed
  WARNING: Partitioning is a write-once setting once it is set to complete.
  Power cycling is required to initialize partitions after set to complete.
mmc bootbus dev boot_bus_width reset_boot_bus_width boot_mode
 - Set the BOOT_BUS_WIDTH field of the specified device
mmc bootpart-resize <dev> <boot part size MB> <RPMB part size MB>
 - Change sizes of boot and RPMB partitions of specified device
mmc partconf dev boot_ack boot_partition partition_access
 - Change the bits of the PARTITION_CONFIG field of the specified device
mmc rst-function dev value
 - Change the RST_n_FUNCTION field of the specified device
   WARNING: This is a write-once field and 0 / 1 / 2 are the only valid values.
mmc setdsr <value> - set DSR register value
```


