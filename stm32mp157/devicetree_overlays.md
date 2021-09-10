# DeviceTree-Overlays

## 编译
* 使用make编译
    ```sh
        cd overlays/

        export STAGING_KERNEL_DIR=/home/user/workdir/linux-toradex  #指定内核的路径

        make or make target
    ```

* 人工编译
    ```sh
        cd overlays/

        cpp -nostdinc -I ../../linux-toradex/arch/arm64/boot/dts -I ../../linux-toradex/include -undef -x assembler-with-cpp verdin-imx8mm_lt8912_overlay.dts verdin-imx8mm_lt8912_overlay.dts.preprocessed

        dtc -@ -Hepapr -I dts -O dtb -i ../../linux/arch/arm/boot/dts/ -o verdin-imx8mm_lt8912_overlay.dtbo verdin-imx8mm_lt8912_overlay.dts.preprocessed
    ```
    or 野火
    ```sh
        # 用的是linux-kernel路径下的 scripts 目录
        ./scripts/dtc/dtc -I dts -O dtb -o xxx.dtbo arch/arm/boot/dts/xxx.dts # 编译 dts 为 dtbo
        ./scripts/dtc/dtc -I dtb -O dts -o xxx.dts arch/arm/boot/dts/xxx.dtbo # 反编译 dtbo 为 dts
    ```
    or
    ```sh
        dtc -I dts -O dtb -@ BB-UART1-00A0.dts > BB-UART1-00A0.dtbo //编译生成.dtbo
        dtc -I dtb -O dts BB-UART1-00A0.dtbo > BB-UART1-00A0.dts    //反编译
    ```


## 部署
* 内核运行状态下加载
    1. 在/sys/kernel/config/device-tree/overlays/下创建一个新目录：
        mkdir /sys/kernel/config/device-tree/overlays/xxx
    2. 将dtbo固件echo到path属性文件中
        echo xxx.dtbo >/sys/kernel/config/device-tree/overlays/xxx/path
        或者将dtbo的内容cat到dtbo属性文件
        cat xxx.dtbo >/sys/kernel/config/device-tree/overlays/xxx/dtbo
    3. 节点将被创建，查看内核设备树
        ls /proc/device-tree
    4. 删除"插件"设备树
        rmdir /sys/kernel/config/device-tree/overlays/xxx

* uboot加载
    修改bootfs的uEnv.txt

