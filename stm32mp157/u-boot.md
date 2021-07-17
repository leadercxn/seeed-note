# U_Boot

## u-boot的 spl 说明
* 参考资料
    1. [uboot-spl编译流程](https://www.kancloud.cn/bjm123456/linux/1877550)

## ST u-boot的使用
* 编译流程
    1. 根据ST的官方U-BOOT目录下的 `README.HOW_TO.txt`
    2. 设置环境变量
        ```shell
            $ source <path to SDK>/environment-setup-cortexa7t2hf-neon-vfpv4-ostl-linux-gnueabi
        ```
    3. 打补丁
        ```
            for p in `ls -1 ../*.patch`; do patch -p1 < $p; done
        ```
    4. 设置环境变量(用来生成 fip.bin文件,指向生成 fip 的路径)
        ```
            export FIP_DEPLOYDIR_ROOT=$PWD/../../FIP_artifacts
        ```
    5. 生成 fiptool 工具，因为Makefile.sdk 文件中有指定 fiptool-stm32
        + 到 tf-a 源码目录下的 <trust-firmware A source dir>/tool/fiptool 下 进行 make , 生成 fiptool,再 cp fiptool fiptool-stm32,把这两可执行文件的路径添加到环境变量PATH中。
    6. 修改 Makefile.sdk 中的 `DEVICE_TREE` 变量，指向对应的对应的设备树文件名
        ```shell
            DEVICE_TREE ?= stm32mp157c-odyssey
        ```
    7. 编译
        ```shell
            make -f ../Makefile.sdk all
        ```

    8. 生成的 fip 集合固件在 FIP_DEPLOYDIR_ROOT 路径下

## uboot 阶段使用命令
* 使用命令加载内核和设备树文件。目前 uImage 镜像在uboot阶段，出现CRC错误，暂还没找到原因
    ```sh
        printenv #打印环境变量，找到 fdt_addr_r=0xc4000000 kernel_addr_r=0xc2000000 内核和设备树加载的 DRAM地址

        fatload mmc 0:4 0xc2000000 zImage   # 从sd卡的第4分区找到zImag文件加载到 DRAM 的 0xc2000000 地址处
        fatload mmc 0:4 0xc4000000 stm32mp157c-odyssey.dtb  # 从sd卡的第4分区找到对应设备树文件加载到 DRAM 的 0xc4000000 地址处
        setenv bootargs 'console=ttySTM0,115200 root=/dev/mmcblk0p5 rootfstype=ext4 rootwait rw'    # root= 指向sdk卡存放文件系统的分区
        bootz 0xc2000000 - 0xc4000000   # 跳转到DRAM中执行
    ```

## npi_common.h 的修改
* npi_common.h 的说明
    1. 就是在uboot阶段，搜索启动盘（例如sd卡）第四分区的文件系统中 uEnv.txt 文件，提取相关的变量信息，作为 bootargs 传递给 kernel。

* 修改

