#  Yocto-Kernel

## linux内核相关的任务
* linux内核相关的任务
    1. do_kernel_menuconfig
        make menuconfig
        ```sh
            bitbake linux-stm32mp -c menuconfig
        ```
    2. do_compile_kernelmodules
        运行构建linux内核模块的步骤。
        ```sh
            bitbake linux-stm32mp -c compile_kernelmodules
        ```
    3. do_diffconfig
        创建一个文件用于保存 do_kernel_configme 任务生成的原始配置与用户使用其他方法所做的更改之间的差异内容，类似于diff命令
    4. do_kernel_checkout
        将解压的内核源码转换为 OpenEmbedded build system 可以使用的形式。
    5. do_kernel_configcheck
        验证 do_kernel_menuconfig 任务生成的配置
    6. do_kernel_configme
        在内核被 do_patch 任务修补后，do_kernel_configme 任务将所有内核配置片段集合到一个配置中，然后又将这些片段传递到相应的内核配置阶段
    7. do_savedeconfig
        创建一个defconfig文件，改文件可用于代替默认的defconfig，保存的defconfig包含默认defconfig与用户使用其他方法（例do_kernel_menuconfig任务）所做更改之间的区别
    8. do_shared_workdir
        编译内核之后，但在编译内核模块之前，此任务将模块构建所需要的文件以及从内核构建生成的文件复制到共享工作目录中。
    9. do_sizecheck
        在构建内核之后，此任务根据 KERNEL_IMAGE_MAXSIZE 变量检查stripped后的内核映像的大小，如果设置了该变量，并且内核大于这个值，在构建过程中，会产生一个警告

## 设备树外添加linux驱动模块
* 直接上demo
    步骤：
    1. copy meta-skeleton/recipes-kernel/hello-mod 该目录到自己的meta layer
    2. 验证,对比bb文件里面的CHKSUM变量值
        ```sh
            md5sum hello-mod/COPYING
        ```
    3. 编译模块
        ```sh
            bitbake hello-mod
        ```
    4. ko 所在目录，但我在st-yocto工程中没见到
        ``` 
            build]$ cd tmp/work/qemux86_64-poky-linux/hello-mod/0.1-r0/
            0.1-r0]$ ls image/ 
            etc  lib  usr
            0.1-r0]$ ls image/lib/modules/5.4.50-yocto-standard/extra/hello.ko 
        ```
    5. 驱动模块添加到image中
        注意以下4个变量
        ```sh
            MACHINE_ESSRNTIAL_EXTRA_RDEPENDS
            MACHINE_ESSRNTIAL_EXTRA_RRECOMMENDS
            MACHINE_EXTRA_RDEPENDS  # 即使镜像中无法包含该模块，也不会导致构建失败
            MACHINE_EXTRA_RRECOMMENDS # 通过将不带.ko扩展名的模块文件名附加到字符串"kernel-module-"中得到
        ```
        
        vim meta/conf/machine/qemux86-64.conf 后添加  
        ` MACHINE_EXTRA_RRECOMMENDS = "kernel-module-snd-ens1370 kernel-module-snd-rawmidi kernel-module-hello" `
    6. 后重新编译，验证驱动

## 利用devtool修改内核
* 步骤
    1. 搞到源码，修改源码
        ```sh
            devtool modify linux-stm32mp    #会在 build/workspace/sources/linux-stm32mp 目录下放置linux内核源码
        ```
    2. 编译打包镜像
        ```
            devtool build linux-stm32mp
            or
            bitbake core-image-sato
        ```
    3. commit 生成patch
        ```sh
            git add -u
            git commit
        ```
    4. 合并到自己的meta layer
        ```sh
            devtool finish linux-stm32mp ../../../meta-stm32mp-odyssey
            # 会在 meta-stm32mp-odyssey/recipes-kernel/linux生成patch文件和bbappend文件
        ```



