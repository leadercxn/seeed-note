# U-boot


## 
* 启动阶段
    1. 相关头文件
        - u-boot/configs/stm32mp13_defconfig
            ```sh
                CONFIG_ARCH_STM32MP=y
                CONFIG_STM32MP13x=y
                CONFIG_ENV_OFFSET=0x480000
                CONFIG_ENV_SECT_SIZE=0x40000
                CONFIG_DEFAULT_DEVICE_TREE="stm32mp135f-dk"
                CONFIG_BOOTCOMMAND="run bootcmd_stm32mp"

                CONFIG_FASTBOOT_BUF_ADDR=0xC0000000
                CONFIG_FASTBOOT_BUF_SIZE=0x02000000
            ```

        - u-boot/include/configs/stm32mp1.h
            ```C
                kernel_addr_r=0xc2000000
                fdt_addr_r=0xc4000000
                fdtoverlay_addr_r=0xc4100000
                scriptaddr=0xc4100000
                pxefile_addr_r=0xc4200000
                splashimage=0xc4300000
                ramdisk_addr_r=0xc4400000
            ```

        - u-boot/include/config_distro_bootcmd.h  boot阶段的环境变量
            ```C
                boot_syslinux_conf=extlinux/extlinux.conf


            ```

* uboot 操作命令
    1. 操作命令
        ```sh
            load mmc 0:4 0xc2000000 uImage
            load mmc 0:4 0xc4000000 stm32mp135f-dk.dtb
            bootm 0xc2000000 - 0xc4000000
        ```

* 分析
    1. u-boot/include/config_distro_bootcmd.h
        BOOTENV_SHARED_MMC  =>  BOOTENV_SHARED_BLKDEV(mmc)
            => mmc_boot = 
                if mmc dev ${devnum};
                    devtype = mmc;
                run scan_dev_for_boot_part;

        BOOT_TARGET_DEVICES(func) => BOOT_TARGET_DEVICES(BOOTENV_DEV)
            => BOOT_TARGET_MMC1(BOOTENV_DEV) => BOOTENV_DEV(MMC, mmc, 0)
                => BOOTENV_DEV_MMC(MMC , mmc 0)  =>  BOOTENV_DEV_BLKDEV(MMC , mmc 0)
                    =>  bootcmd_mmc0 = devnum = 0;          //提取出 devnum
                        run mmc_boot
                => BOOTENV_DEV_MMC(MMC , mmc 1)  =>  BOOTENV_DEV_BLKDEV(MMC , mmc 1)
                    =>  bootcmd_mmc1 = devnum = 1;
                        run mmc_boot

        BOOTENV_BOOT_TARGETS
            => "boot_targets=" BOOT_TARGET_DEVICES(BOOTENV_DEV_NAME)
                => boot_targets = bootcmd_mmc1
                                  bootcmd_mmc0