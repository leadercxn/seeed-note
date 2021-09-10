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
                            -n 3:1058:5153   -c 3:fip    \
                            -n 4:5154:136225 -c 4:bootfs    \
                            -n 5:136226:     -c 5:rootfs  \
                            -p /dev/sdb

                # 设置BIOS的旧分区
                sudo sgdisk -A 4:set:2 /dev/sdb

                # 查看设备所有分区的名字
                sudo sgdisk -p /dev/sdb
            ```
    3. 存入 u-boot-spl.stm32 或 tf-a-xxxx.stm32 、u-boot.img
            ```shell
                sudo dd if=./u-boot/spl/u-boot-spl.stm32 of=$/dev/sdb1
                sudo dd if=./u-boot/spl/u-boot-spl.stm32 of=$/dev/sdb2
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

