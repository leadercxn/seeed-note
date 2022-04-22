# Start_package

## 指导wiki
* 指导wiki
    1. [ST官网指导wiki](https://wiki.stmicroelectronics.cn/stm32mpu/wiki/STM32MP15_Discovery_kits_-_Starter_Package)

* 烧录操作
    ```
        STM32_Programmer_CLI -c port=usb1 -w flashlayout_st-image-weston/trusted/FlashLayout_sdcard_stm32mp157c-dk2-trusted.tsv
    ```

## tsv 文件
* 语法
    ```sh
        - : no action
        P : update = program the partition or the flash device
        PE : do not update (also EP) : allow the GPT partitioning with empty partition for the block device but equivalent to '-' for RAW flash device
        PD : delete and update (also DP)
        PDE : delete and keep empty (also PED / DPE / DEP / EPD / EDP)
    ```

## 踩过的坑
* 踩过的坑
    1. 第一次使用的时候，官方指导步骤里，有个命令行
        ```shell
            STM32_Programmer_CLI -c port=usb1 -w flashlayout_st-image-weston/trusted/FlashLayout_sdcard_stm32mp157c-dk2-trusted.tsv
        ```
        该命令行可在windows、ubuntu环境下进行操作，但直接操作的话，会出现 error,mmc0 fail类似的错误提示，故需要修改对应    .tsv文件 ，修改如下
        ```
            PD	0x06	ssbl	Binary	mmc0	0x00084400	bootloader/u-boot-stm32mp157c-dk2-trusted.stm32
            修改为
            P	0x06	ssbl	Binary	mmc0	0x00084400	bootloader/u-boot-stm32mp157c-dk2-trusted.stm32
        ```
        取消D ，delete的操作




