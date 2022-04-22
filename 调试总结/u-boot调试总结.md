# U-BOOT

## u-boot
* mmc
    mmc list                                // 查看当前系统挂在的 mmc 设备
    mmc dev 1                               // 切换到mmc1设备
    ls mmc 1:2 /boot                        // 查看mmc1设备的2分区 /boot 目录下的文件
    load mmc 1:2 0x82000000  /boot/Image    // 把mmc1设备2分区下的 /boot/Image 文件 加载到 0x82000000

    part list mmc 0                         // 列出mmc0的所有分区