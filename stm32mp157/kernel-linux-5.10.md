# Kernel-5.10

## ODYSSEY-NPi 板子
* 操作步骤流程
    1. 根据ST官方的 LINUX-5.10的目录下的 `README.HOW_TO.txt`
    2. 设置环境变量
        ```shell
            $ source <path to SDK>/environment-setup-cortexa7t2hf-neon-vfpv4-ostl-linux-gnueabi
        ```
    3. 安装相应的工具
        ```
            sudo apt-get install u-boot-tools
            sudo apt-get install libyaml-dev
        ```
    4. 打补丁
        ```
            for p in `ls -1 ../*.patch`; do patch -p1 < $p; done
        ```
    
    5. 修改设备文件，让kernel支持sd卡。在 `stm32mp157c-odyssey.dts` 文件修改 sdmmc1 节点信息
        ```dts
            &sdmmc1 {
                pinctrl-names = "default", "opendrain", "sleep";
                pinctrl-0 = <&sdmmc1_b4_pins_a>;
                pinctrl-1 = <&sdmmc1_b4_od_pins_a>;
                pinctrl-2 = <&sdmmc1_b4_sleep_pins_a>;
                cd-gpios = <&gpioi 3 (GPIO_ACTIVE_LOW | GPIO_PULL_UP)>;     # ODYSSEY的IO不同
                disable-wp;
                st,neg-edge;
                bus-width = <4>;
                vmmc-supply = <&v3v3>;
                status = "okay";
            };
        ``

    6. 修改配置文件
        + 个人使用 新建build目录来放编译过程中产生的中间文件和最终文件的方式。（方便区分不同的配置生成对应的产物）
            ```shell
                export build_dir=build  #这样方便修改目录

                mkdir -p ../$build_dir
                make ARCH=arm O="$PWD/../$build_dir" multi_v7_defconfig fragment*.config

                for f in `ls -1 ../fragment*.config`; do scripts/kconfig/merge_config.sh -m -r -O $PWD/../$build_dir $PWD/../$build_dir/.config $f; done
                
                yes '' | make ARCH=arm oldconfig O="$PWD/../$build_dir"

                make ARCH=arm uImage vmlinux dtbs LOADADDR=0xC2000040

                make ARCH=arm modules

                make ARCH=arm INSTALL_MOD_PATH="$PWD/../build/install_artifact" modules_install

                mkdir -p $PWD/../build/install_artifact/boot/

                cp $PWD/../build/arch/arm/boot/uImage $PWD/../build/install_artifact/boot/
                cp $PWD/../build/arch/arm/boot/zImage $PWD/../build/install_artifact/boot/
                cp $PWD/../build/arch/arm/boot/dts/st*.dtb $PWD/../build/install_artifact/boot/
            ```

            生成的文件在：
                $PWD/install_artifact/boot/uImage
                $PWD/install_artifact/boot/zImage       # 建议使用这个
                $PWD/install_artifact/boot/<stm32-boards>.dtb

        + 在kernel-source目录下进下编译，也可以。
            ```shell
                ...
            ```
    
    7. 把生成的设备树，和kernel镜像文件放到 sd 的第四分区 bootfs
        ```shell
            cp -r $build_dir/install_artifact/boot/*    /media/$USER/bootfs/ 
        ```
    8. 把生成的 kernel modules 拷贝到 sd 卡的文件系统分区
        ```shell
            rm -rf $build_dir/install_artifact/lib/modules/<kernel version>/source  $build_dir/install_artifact/lib/modules/<kernel version>/build

            find $build_dir/install_artifact/ -name "*.ko" | xargs $STRIP --strip-debug --remove-section=.comment --remove-section=.note --preserve-dates

            sudo cp -r lib/modules/* /media/$USER/rootfs/lib/modules/
        ```
        后在板子上执行
        ```
            depmod -a
            sync
            reboot
        ```