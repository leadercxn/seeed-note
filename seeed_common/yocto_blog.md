# Yocto开发讲解系列


## blog 详解目录
* [Yocto开发篇](https://blog.csdn.net/fulinus/category_10445379.html)


## yocto 基础
* 构建项目的流程
    1. Fetch
    2. Extract
    3. Patch
    4. Configure
    5. Build
    6. Install
    7. Package
* 平台层： 例：meta-st
* 特征层： 例：meta-qt5

## 文件功能描述
* .conf
    定义了控制 OpenEmbedded 生成过程的各种配置变量
    1. build/conf/local.conf
        ```sh
            DL_DIR  源码下载目录，like:build/downloads
            SSTATE_DIR  共享状态目录  like: build/sstate-cache
            TMPDIR  编译输出目录  like:build/tmp
            MACHINE 目标machine
            DISTRO  distribution策略
            PACKAGE_CLASSES 包装格式
            SDKMACHINE  SDK目标体系架构， like: SDKMACHINE ?= "i686"
            EXTRA_IMAGE_FEATURES    额外Image包
        ```
    2. build/conf/bblayers.conf
        告诉了Bitbake要在build期间使用哪些layers

    3. 相关说明
        + Bitbake target 构建时，bitbake 会对配置文件按顺序加载，指定顺序是 site.conf,auto.conf,local.conf,所以在这些文件中定义的变量会以最后一个文件的值为准
        + /build/conf/local.conf 文件中的 MACHINE 变量，like: stm32mp1 ,编译的时候，会根据这个变量查找到对应 meta-st-stm32mp/conf/machine/stm32mp1.conf 配置文件，并按照这个文件中的规则构建
        + meta-xx/conf/layer.conf
            * 说明
                ```sh
                    poky]$ cat meta-mylayer/conf/layer.conf
                    ...
                    # We have recipes-* directories, add to BBFILES
                    BBFILES += "${LAYERDIR}/recipes-*/*/*.bb \
                                ${LAYERDIR}/recipes-*/*/*.bbappend"
                    ...
                ```
                所以 .bb or .bbappend 文件只能放在 receips-xx 后的第二级目录下

* .bb
    包含单个软件的信息，包括源码下载的位置，来源。 补丁文件， 应用的特殊配置选项、如何编译、打包编译的输出文件
* .bbclass
    类文件，用于配方文件之间的共享信息。

## OpenEmbedded构建系统概念
* 通用的工作流程分这三层
    1. Distro Layer
        为自己的发行版提供策略配置，like: meta-st-openstlinux。 一般包含 meta-xxx/conf/distro/distro.conf

    2. Software Layer
        为构建期间使用的其他软件包提供元数据。此层不包含特定的发行版或machine信息，即与平台无相关的软件。比如 busybox/Qt等底单房软件包

    3. BSP Layer
        与硬件相关的layer。 包含 bootloader、kernel、驱动等。此层为特定的 machine生成image或SDk。like: meta-st-stm32mp.一般包含 meat-xxx/conf/machine/xxx

* Recipes
    recipes配方是要构建软件和镜像的逻辑但愿，是构建一个项目的脚本
    ```sh
        bitbake-layers show-recipes # 可查看yocto有哪些recipes
    ```
    * 相关的 recipes 说明
        recipes-bsp         —任何带有特定硬件或硬件配置信息链接的内容
        recip- connectivity -与其他设备通信相关的库和应用程序
        recipes-core        —构建包括常用依赖项在内的基本工作Linux映像所需要的内容
        recipes-devtools    —主要用于构建系统的工具(但也可以用于目标)
        recipes-extended    -与内核中的替代方案相比，这些应用程序虽然不是必不可少的，但却可以添加一些特性。可能需要完整的工具功能。所有与GTK+应用程序框架相关的内容
        recipes-graphics    -X和其他图形相关的系统库
        recipes-kernel      —具与内核有强依赖性的通用应用程序或者库
        recipes-multimedia  -音频、图像和视频的编解码器和支持应用程序
        recipes-rt          —提供用于使用和测试PREEMPT_RT内核的包和映像食谱
        recipes-sato        -参考UI/UX，相关的应用程序和配置
        recipes-support     —其他食谱使用的食谱，但没有直接包含在图片中




## 技巧
* 搜索文件 && 查找关键字
    1. 安装工具,此工具安装超久，3h以上
        ```sh
            sudo apt-get install mlocate
        ```
    2. 在 yocto 工程目录下生成数据库
        ```sh
            sudo updatedb -U . -o yocto_updatedb
        ```
    3. 搜索文件
        ```sh
            locate -d  yocto_updatedb  linux-stm32mp1_5.10.bb
        ```
        可以把  ` alias yocto_find ='locate -d  AbsolutePath/yocto_updatedb'  ` 放到 ~/.bashrc 文件里
    4.  查找某些文件中的关键字
        ```sh
            yocto_find *.bb *.bbappend *.inc | xargs grep -in KERNEL_EXTRA_FEATURES
        ```
    5. 过滤某个文件夹
        ```sh
            yocto_find  linux-stm32mp1_5.10.bb | grep -v "build/"
        ```
    6. 搜索结果输出到某个文件
        ```sh
            for c in `mylocate *.bb *.bbappend *.inc *.conf *.bbclass | grep -v "poky/build"`;do echo $c >> allbbandinc;cat $c >> allbbandinc;done
        ```
    7. 有待验证,假如目录中的文件发生改变，是否再次执行 `updatedb -U . -o yocto_updatedb`命令



## 实操
* 安装执行程序到文件系统中
    1. 找到生成文件系统的 xxx.bb 文件，在自己的 meta-xx 添加对应目录下的 xxx.bbappend 文件
        + demo
            1. st原生的文件系统target是 meta-st-openstlinux/recipes-st/image/st-image-core.bb
            2. 在自己的 meta-xxx 中添加对应的 recipes-st/image/st-image-core.bbappend 文件
                ```
                    IMAGE_INSTALL += "helloworld-m"
                    IMAGE_INSTALL += "helloworld-a"
                    IMAGE_INSTALL += "helloworld-c"
                ```
                添加自己的目标文件
 
## Bitbake 工具
1. 

