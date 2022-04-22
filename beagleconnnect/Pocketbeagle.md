# beagleblack

## Operation in 基本配置
* 镜像下载
    1. [镜像下载](https://rcn-ee.net/rootfs/)

* 系统配置
    1. vim /boot/uEnv.txt
        uboot_overlay_addr0=/lib/firmware/dev-USB-PWR-CTL-00A1.dtbo #打开电源USB电源控制引脚
    2. connmanctl
        后续输入命令:     services
                        agent on
                        connect wifi_60ee5c6699db_5354552d4545_managed_psk（对应要连接的wifi后面的名字）
                        设置要连接wifi的密码
                        quit
    3. 升级内核版本 和 安装对应版本的头文件
        sudo apt install linux-image-5.10.59-ti-r16 linux-headers-5.10.59-ti-r16
    4. 扩大SD卡的分区
        sudo ./opt/scripts/tools/grow_partition.sh 重启生效

* 更新
    1. 软件包的更新
          install

    2. 头文件安装，版本要对应【安装5.10内核版本即可】
        sudo apt install linux-headers-4.19.94-ti-r66
    
## Greybus 的调试
* greybus 源码获取
    1. git clone https://github.com/jadonk/greybus
    2. git checkout -b beagleconnect

    3. git clone https://github.com/finikorg/wpanusb.git
    4. cd wpanusb

    5. git clone https://github.com/linux-wpan/wpan-tools.git
    6. cd wpan-tools
    7. export LC_ALL=C
    8. ./autogen.sh
    9. sudo apt-get install libnl-3-dev libnl-genl-3-dev 
    10. ./configure CFLAGS='-g -O0' --prefix=/usr --sysconfdir=/etc --libdir=/usr/lib
    11. make && sudo make install
    12. iwpan list
    12. sudo iwpan phy phy0 set channel 0 1
    
    12. sudo ip link add link wpan0 name lowpan0 type lowpan
    13. sudo ip link set wpan0 up
    14. sudo ip link set lowpan0 up
    15. ip a show lowpan0
    16. ping6 -I lowpan0 ff02::1

    20. sudo ip link set lowpan0 down
    21. sudo ip link set wpan0 down

* 集成greybus后的镜像
    1. sudo beagleconnect-start-gateway
    2. iio_info
    3. dmesg

* wpanusb 源码获取
    1. 
    2. 使用驱动
        sudo cp wpanusb.ko /lib/modules/xxxversion/kernel/drivers/net/ieee802154/
        sudo depmod -a
        sudo modprobe wpanusb
        ifconfig -a

