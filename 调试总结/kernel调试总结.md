# kernel

## driver
* 设备树打印
    dtc -I fs /proc/device-tree > dts.txt

* i2cdetect && i2ctool
    1. i2cdetect
        ```sh
            i2cdetect -l  //列举i2c总线
            i2cdetect -y -r 1   //用 i2c1 扫描总线上的期间
        ```
    2. i2cget && i2cset
        demo: 要读取 ov5640 的ID寄存器 0x300a => 0x5640
        ```
            i2cset -y -f 0 0x3c 0x30 0x0a   //首先 设置要读取的地址 0x300a

            i2cget -y -f 0 0x30
                => 0x56
            i2cget -y -f 0 0x0a
                => 0x40
        ```


* spidev_test
    1. 可通过如下，在spi节点下添加虚拟设备，添加后，可在系统下见到 /dev/spidev0.0
        ```dts
            spidev0: spidev@0{
                compatible = "spidev";
                reg = <0>;	/* CE0 */
                #address-cells = <1>;
                #size-cells = <0>;
                spi-max-frequency = <125000000>;
            };

            spidev1: spidev@1{
                compatible = "spidev";
                reg = <1>;	/* CE1 */
                #address-cells = <1>;
                #size-cells = <0>;
                spi-max-frequency = <125000000>;
            };
        ```

    2. 把spi的 MOSI 和 MISO 短接,执行如下命令，可以实现写啥，读啥
         ./spidev_test -D /dev/spidev0.0 -v -p  hello_world

    3. 

* serial
    1. 系统路径 /sys/class/tty, 可通过 ls -l 命令查看soc的硬件串口是映射到哪个tty上
    2. 通过minicom来调试


* pwm 调试
    1. cd /sys/class/pwm/pwmchip0/
    2. echo 0 > export  //这是选择通道CH1的打开方式， 其他通道例如CH3,就 echo 2 > export 
    3. cd pwm0/
    4. echo 10000 > period
    5. echo 5000 > duty_cycle
    6. echo 1 > enable

* 更新驱动后，或更内核源码后，要记得
    make ARCH=arm modules O="$PWD/../build"
    make ARCH=arm INSTALL_MOD_PATH="$PWD/../build/install_artifact" modules_install O="$PWD/../build"

    生成的 install_artifact/lib/modules/kernel-version 目录替换掉系统的 /lib/modules/kernel-version

    开机后 

    depmod -a   //更新驱动依赖列表
    sync
    reboot


* 查看系统时钟树
    1. cat /sys/kernel/debug/clk/clk_summary


* 显示屏
    1. 命令: modetest
        [modetest 命令](https://blog.csdn.net/u012839187/article/details/103507833)

        demo:在 ST 平台上
        ```sh
            modetest -M stm -s connect_id:480x272
        ```
    2. 进入命令行界面的两种方法
        系统启动后: init 3  => 进入图形界面: init 5
        or
        vim /etc/inittab => id:3:initdefault:   # 命令行界面
        vim /etc/inittab => id:5:initdefault:   # 图形界面





## tracepoint 的打印方法
* [使用Linux内核的Tracepoint的体验](https://blog.csdn.net/dog250/article/details/112058189)
* [Trace - 一文读懂tracepoint](https://blog.csdn.net/rikeyone/article/details/116057261?spm=1001.2101.3001.6650.6&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-6.queryctr&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-6.queryctr&utm_relevant_index=8)

## KGDB的调试方式
* [KGDB调试](http://blog.chinaunix.net/uid-29401328-id-4890452.html)
* [使用kdb和kgdb调试Linux内核(1)](https://www.cnblogs.com/ainima/p/6330785.html)
* [imx6 KGDB调试方法总结(光谷王凯的博客)](https://www.it610.com/article/1293974852646019072.htm)
