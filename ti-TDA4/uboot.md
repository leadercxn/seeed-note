# U-boot


## 
* 启动阶段
    1. 相关头文件
        - u-boot/configs/j721e_evm_a72_defconfig
            ```sh
                CONFIG_BOOTCOMMAND="run findfdt; run envboot; run init_${boot}; run boot_rprocs; run get_kern_${boot}; run get_fdt_${boot}; run get_overlay_${boot}; run run_kern"  #启动脚本 boot_cmd

                CONFIG_ENV_SIZE=0x20000 #128K

                CONFIG_DEFAULT_DEVICE_TREE="k3-j721e-beagleboneai64"    # 设备树

                CONFIG_SPL_TEXT_BASE        =   0x80080000
                CONFIG_SPL_STACK_R_ADDR     =   0x82000000
                CONFIG_SPL_SIZE_LIMIT       =   0x2D000
                CONFIG_SPL_LOAD_FIT_ADDRESS =   0x81000000

                CONFIG_SYS_DFU_DATA_BUF_SIZE    =   0x20000
                CONFIG_SYS_DFU_MAX_FILE_SIZE    =   0x800000

                CONFIG_FASTBOOT_BUF_ADDR        =   0x82000000
                CONFIG_FASTBOOT_BUF_SIZE        =   0x2F000000
            ```
        - u-boot/include/configs/j721_evm.h
            ```C
                boot=mmc
                mmcdev=1
                bootpart=1:2
                bootdir=/boot
                rd_spec=-
            ```
        - include/configs/ti_armv7_common.h
            ```C
                #define DEFAULT_LINUX_BOOT_ENV
                        loadaddr=0x82000000
                        kernel_addr_r=0x82000000
                        fdtaddr=0x88000000
                        dtboaddr=0x89000000
                        fdt_addr_r=0x88000000
                        rdaddr=0x88080000
                        ramdisk_addr_r=0x88080000
                        scriptaddr=0x80000000
                        pxefile_addr_r=0x80100000
                        bootm_size=0x10000000
                        boot_fdt=try
            ```

* 分析
    1. u-boot/include/configs/j721_evm.h
        ```C
            run findfdt:
                $board_name = BBONEAI-64-B0-
                name_fdt = k3-j721e-beagleboneai64.dtb
                fdtfile = k3-j721e-beagleboneai64.dtb

            run get_kern_${boot} => run get_kern_mmc
                load mmc ${bootpart} ${loadaddr}  ${bootdir}/${name_kern}
                    => load mmc 1:2 0x82000000  /boot/Image

            run get_fdt_${boot} => run get_fdt_mmc
                load mmc ${bootpart} ${fdtaddr} ${bootdir}/${name_fdt}
                    => load mmc 1:2 0x88000000  /boot/k3-j721e-beagleboneai64.dtb

            run get_overlay_${boot} => run get_overlay_mmc
                fdt address ${fdtaddr}; => fdt address 0x88000000
                fdt resize 0x100000;    => 
                for overlay in $name_overlays;
                    do;
                        load mmc ${bootpart} ${dtboaddr} ${bootdir}/${overlay} && fdt apply ${dtboaddr};
                            => load mmc 1:2 0x89000000 /boot/${overlay} && fdt apply 0x89000000             //fdt apply 应用overlay
                    done;


            run run_kern
                booti ${loadaddr} ${rd_spec} ${fdtaddr}
                    => booti 0x82000000 - 0x88000000

            name_kern=Image
            console=ttyS2,115200n8
            args_all=setenv optargs earlycon=ns16550a,mmio32,0x02800000 ${mtdparts}
            run_kern=booti ${loadaddr} ${rd_spec} ${fdtaddr}    //booti专门用来启动ARM64的kernel image
        ```
        





