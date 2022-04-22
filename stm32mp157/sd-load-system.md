# Seeed-ODYSSEY-STM32MP157-NPi

## TechForum 平台的教程
* 教程链接
    [Getting Start with ODYSSEY](https://forum.digikey.com/t/debian-getting-started-with-the-odyssey-stm32mp157c/12838#ODYSSEY-STM32MP157C-Availability)

* 通用性操作，基于sd卡
    1. 假设 u-boot-spl.stm32 或 tf-a-xxxx.stm32 、u-boot.img、kernel_zImage、rootfs.tar.gz 等文件已准备好
    2. 先针对sd进行操作(这些操作需 sd没在系统上有挂载点)
        + 擦除sd卡的分区信息表 partition table/labels 信息
            ```shell
                sudo dd if=/dev/zero of=/dev/sdb bs=1M count=10
            ```
        + 创建新的信息分区信息表
            ```shell
                # 擦除先前的格式
                sudo sgdisk -o /dev/sdb

                # 重新分区并命名分区的名字
                # fsbl1 和 fsbl2 可放入同一个固件，fsbl2 是 fsbl1损坏时的 备用固件
                # 注意：假如使用 tf-a 的固件作为 fsbl1, fsbl2 的启动固件，第三分区的命名要使用 "fip"（代码中有要求，可以在代码中修改该分区的命名要求）
                #       假如使用 u-boot-spl作为 fsbl1，fsbl2 的启动固件，第三分区的命名建议叫 "ssbl"。
                #       u-boot-spl 是否绑定第三分区的"ssbl"的名字，这个暂还没验证。
                sudo sgdisk --resize-table=128 -a 1 \
                            -n 1:34:545      -c 1:fsbl1   \
                            -n 2:546:1057    -c 2:fsbl2   \
                            -n 3:1058:5153   -c 3:ssbl    \
                            -n 4:5154:136225 -c 4:boot    \
                            -n 5:136226:     -c 5:rootfs  \
                            -p /dev/sdb

                sudo sgdisk --resize-table=128 -a 1 \
                            -n 1:34:545      -c 1:fsbl1   \
                            -n 2:546:1057    -c 2:fsbl2   \
                            -n 3:1058:9249   -c 3:fip    \
                            -n 4:9250:136225 -c 4:bootfs    \
                            -n 5:136226:     -c 5:rootfs  \
                            -p /dev/sdb

                # ST135
                sudo sgdisk --resize-table=128 -a 1           \
                            -n 1:34:545      -c 1:fsbl1       \
                            -n 2:546:1057    -c 2:fsbl2       \
                            -n 3:1058:9249   -c 3:fip         \
                            -n 4:9250:140321 -c 4:bootfs      \
                            -n 5:140322:173089 -c 5:vendorfs  \
                            -n 6:173090:10658849  -c 6:rootfs  \
                            -n 7:10658850:   -c 7:userfs \
                            -p /dev/sdb

                # 设置BIOS的旧分区
                sudo sgdisk -A 4:set:2 /dev/sdb

                # 查看设备所有分区的名字
                sudo sgdisk -p /dev/sdb

                # 查看文件分区
                sudo fdisk -l sys.img

                # 查看设备所有分区的文件系统类型
                lsblk -f /dev/sdb
            ```
    3. 存入 u-boot-spl.stm32 或 tf-a-xxxx.stm32 、u-boot.img
            ```shell
                sudo dd if=./u-boot/spl/u-boot-spl.stm32 of=/dev/sdb1
                sudo dd if=./u-boot/spl/u-boot-spl.stm32 of=/dev/sdb2
                sudo dd if=./u-boot/u-boot.img of=/dev/sdb3
            ```
    4. 分区格式处理
            ```shell
                sudo mkfs.vfat -F 16 -n BOOT /dev/sdb4  #用来放内核相关的镜像文件
                sudo mkfs.ext4 -L rootfs /dev/sdb5  # 用来放解压之后的文件系统
            ```
    5. sd卡分区挂载到系统上
        ```
            sudo mkdir -p /media/boot/
            sudo mkdir -p /media/rootfs/
            
            sudo mount /dev/sdb4 /media/boot/
            sudo mount /dev/sdb5 /media/rootfs/
        ```
    6. 内核文件的烧写
        ```shell
            # 把用到的设备树镜像.dtb 拷贝到内核分区的目录上
            sudo tar xfvo dtbs.tar.gz -C /media/boot/dtbs/${kernel_version}/
            or
            cp xxxx.dtb /media/boot/dtbs/${kernel_version}/

            sudo cp -v kernel_zImage /media/boot/zImage
        ```
    7. 文件系统的烧写
        ```shell
            # 信息文本
            sudo sh -c "echo 'uname_r=${kernel_version}' >> /media/boot/uEnv.txt"
            sudo sh -c "echo 'dtb=stm32mp157c-seeed-npi.dtb' >> /media/boot/uEnv.txt"

            sudo tar xfvp ubuntu-rootfs.tar -C /media/rootfs/

            # 把相关驱动.ko 文件拷贝到文件系统
            sudo tar xfv ./armv7-lpae-multiplatform/deploy/${kernel_version}-modules.tar.gz -C /media/rootfs/
            or
            sudo cp xxx.ko xxx.ko /media/rootfs


            sudo chown root:root /media/rootfs/
            sudo chmod 755 /media/rootfs/
        ```
    8. 真正把数据下载到sd卡，卸载sd卡
        ```
            sync

            sudo umount /media/boot
            sudo umount /media/rootfs
        ```


# 制作 .img 镜像
* 步骤
    1. 新建文件
        $ touch test.img
    2. 生成空镜像
        $ sudo dd if=/dev/zero of=test.img bs=1M count=8192
    3. 擦出格式
        $ sudo sgdisk -o test.img
    4. 分区
        $ sudo sgdisk --resize-table=128 -a 1           \
                            -n 1:34:545      -c 1:fsbl1       \
                            -n 2:546:1057    -c 2:fsbl2       \
                            -n 3:1058:9249   -c 3:fip         \
                            -n 4:9250:140321 -c 4:bootfs      \
                            -n 5:140322:173089 -c 5:vendorfs  \
                            -n 6:173090:10658849  -c 6:rootfs  \
                            -n 7:10658850:   -c 7:userfs \
                            -p test.img
    5. 用来连接 img 镜像文件和 loopX回环设备
        $ sudo losetup /dev/loop21 test.img
    6. 挂载虚拟磁盘
        $ sudo kpartx -av /dev/loop21
        此时会有打印
            add map loop21p1 (253:0): 0 512 linear 7:21 34
            add map loop21p2 (253:1): 0 512 linear 7:21 546
            add map loop21p3 (253:2): 0 8192 linear 7:21 1058
            add map loop21p4 (253:3): 0 131072 linear 7:21 9250
            add map loop21p5 (253:4): 0 32768 linear 7:21 140322
            add map loop21p6 (253:5): 0 10485760 linear 7:21 173090
            add map loop21p7 (253:6): 0 6118333 linear 7:21 10658850
        对应的映射分区在
            /dev/mapper/
    7. 格式化分区
        $ sudo mkfs.vfat -F 16 -n bootfs  /dev/mapper/loop21p4 
            # sudo mkfs.fat  -n bootfs  /dev/mapper/loop21p4 
            # sudo mkfs.ext4 -L bootfs  /dev/mapper/loop21p4
        $ sudo mkfs.ext4 -L vendorfs  /dev/mapper/loop21p5
        $ sudo mkfs.ext4 -L rootfs  /dev/mapper/loop21p6
        $ sudo mkfs.ext4 -L userfs  /dev/mapper/loop21p7
    8. 烧录镜像
        $ sudo dd if=tf-a.stm32 of=/dev/mapper/loop21p1
        ...
    9. 卸载虚拟磁盘
        $sudo kpartx -d test.img
    10. 断开 img 镜像文件和 loopX回环设备
        $sudo losetup -d /dev/loop21

