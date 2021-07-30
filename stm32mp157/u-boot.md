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


## 踩坑
1. .config 文件配置的 CONFIG_BOOTCOMMAND , 可追溯到 <uboot source dir>/configs/stm32mp15_trusted_defconfig 文件
    ```sh
        CONFIG_BOOTCOMMAND = "run bootcmd_stm32mp"
    ```

2.  要修改 <uboot source dir>/include/configs/stm32mp1.h
    + 取消自动保存环境变量的设定,建议这么做。
        ```c
            "run env_check;" \

            "env_check=if env info -p -d -q; then env save; fi\0" \
        ```
    + 添加宏
        ```c
            #define BOOT_TARGET_LEGACY_MMC0(func)  func(LEGACY_MMC,legacy_mmc,0)
            #define BOOT_TARGET_LEGACY_MMC1(func)  func(LEGACY_MMC,legacy_mmc,1)
        ```
    + 改变环境变量的目标值
        ```c
            #define BOOT_TARGET_DEVICES(func)	\
                BOOT_TARGET_MMC1(func)		\
                BOOT_TARGET_UBIFS(func)		\
                BOOT_TARGET_MMC0(func)		\
                BOOT_TARGET_MMC2(func)		\
                BOOT_TARGET_PXE(func)
        ```
3. 了解 <uboot source dir>/include/config_distro_bootcmd.h 文件，是uboot变量和命令的设置

4. 由于在环境变量中增加了自定义的命令,所以得在 <u-boot dir>/configs/stm32mp_xx_defconfig 文件中增大 环境变量的大小修改为 `CONFIG_ENV_SIZE=0x4000`

