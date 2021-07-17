# ARM Trusted-Firmware  (TF-A)

## ODYSSEY-STM32MP157c-NPi，后称 NPi
* BL2固件的生成
    因为 ODYSSEY-STM32MP157c-NPi 在电源管理IC使用的I2C2与ST官方的接口不一样。 为了让 NPi 板子支持 tf-a 的固件，tf-a 源码得需做如下修改
    + [PMIC hardware components 参考链接](https://wiki.stmicroelectronics.cn/stm32mpu/wiki/PMIC_hardware_components#Support_in_Cortex-A7_Secure)
    + 附加：
        1. I2C2接口使用的GPIO需在设备树文件中添加，在 tf-a-stm32mp-2.4.R1-R0 源码目录上的 `fdts/stm32mp15-pinctrl.dtsi`文件中，添加如下节点
            ```dts
                &pinctrl {
                    ...
                        i2c2_pins_a: i2c2-0 {
                            pins {
                                pinmux = <STM32_PINMUX('H', 4, AF4)>, /* I2C2_SCL */
                                        <STM32_PINMUX('H', 5, AF4)>; /* I2C2_SDA */
                                        bias-disable;
                                        drive-open-drain;
                                        slew-rate = <0>;
                                    };
                            };
                    ...
                }
            ```
    + 后编译即可生成 对应的 tf-a-xxxx.stm32 的烧录固件，该固件放在sd卡的 fsbl1 和 fsbl2分区


* fip的固件生成
    + sd 卡的第三个分区必须命名为 fip ,在tf-a源码中有要求。 
    + [How to configure TF-A FIP fip固件生成链接](https://wiki.stmicroelectronics.cn/stm32mpu/wiki/How_to_configure_TF-A_FIP)
        + 例如，利用 fiptool 工具对多个.dtb 和 .bin文件进行镜像打包
            ```shell
                fiptool create --fw-config fw-config.dtb \
                        --hw-config u-boot.dtb \
                        --tos-fw-config bl32.dtb \
                        --tos-fw bl32.bin \ 
                        --nt-fw u-boot-nodtb.bin \
                        fip.bin
                或
                fiptool create --fw-config  stm32mp157c-seeed-npi-fw-config-trusted.dtb \
                        --hw-config  u-boot-stm32mp157c-dk2-trusted.dtb \
                        --tos-fw-config  stm32mp157c-seeed-npi-bl32.dtb \
                        --tos-fw  tf-a-bl32-stm32mp15.bin  \
                        --nt-fw  u-boot-nodtb-stm32mp15.bin \
                        seeed-use-dk2-fip.bin

                fiptool create --fw-config  stm32mp157c-seeed-npi-fw-config-trusted.dtb \
                        --hw-config   stm32mp157c-odyssey.dtb \
                        --tos-fw-config  stm32mp157c-seeed-npi-bl32.dtb \
                        --tos-fw  tf-a-bl32-stm32mp15.bin  \
                        --nt-fw  u-boot-nodtb.bin \
                        odyssey-fip.bin
            ```
        + 假如在调试过程中，修改了 BL32相关源码，可通过 fiptool 工具针对性修改对应的 fip.bin文件
            ```
                fiptool update --tos-fw <tfa_path>/bl32.bin fip.bin
            ```
            然后把新生成的 fip.bin文件 dd 到 sd 卡的 fip 分区
    + fip认知：
        1. fip.bin文件就是把 BL32,BL33的固件(.bin)和他们对应的设备树固件(.dtb)打包成一个固件
            + BL32 是负责安全，认证相关的，要么是 SP_MIN,要么是 OP-TEE
            + BL33 是u-boot.bin


## 踩坑笔记
* STM32MP157的ROM固件，对 tf-a 的镜像组成结构有要求
    + [TF-A BL2 overview](https://wiki.stmicroelectronics.cn/stm32mpu/wiki/TF-A_BL2_overview)


