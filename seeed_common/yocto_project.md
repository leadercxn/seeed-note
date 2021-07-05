# yacto project

## yocto 构建工程说明
* yocto 构建工程的一般步骤 在ubuntu环境下
    1. 安装必要的依赖
        ```shell
            sudo apt-get update
            sudo apt-get upgrade
            sudo apt-get install gawk wget git-core diffstat unzip texinfo gcc-multilib build-essential chrpath socat cpio python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev pylint3 pylint xterm
            sudo apt-get install make xsltproc docbook-utils fop dblatex xmlto
            sudo apt-get install libmpc-dev libgmp-dev
            sudo apt-get install libncurses5 libncurses5-dev libncursesw5-dev libssl-dev linux-headers-generic u-boot-tools device-tree-compiler bison flex g++ libyaml-dev libmpc-dev libgmp-dev
            sudo apt-get install coreutils bsdmainutils sed curl bc lrzsz corkscrew cvs subversion mercurial nfs-common nfs-kernel-server libarchive-zip-perl dos2unix texi2html diffstat libxml2-utils

            git config --global user.name "Your Name"
            git config --global user.email "you@example.com"
            git config --list
        ```
        or
        ```shell
            sudo apt-get install gawk wget git-core
            sudo apt-get install diffstat unzip texinfo gcc-multilib
            sudo apt-get install build-essential chrpath socat libsdl1.2-dev
            sudo apt-get install libsdl1.2-dev
            sudo apt-get install xterm sed cvs
            sudo apt-get install subversion coreutils texi2html
            sudo apt-get install docbook-utils python-pysqlite2
            sudo apt-get install help2man make gcc g++
            sudo apt-get install desktop-file-utils
            sudo apt-get install libgl1-mesa-dev libglu1-mesa-dev mercurial
            sudo apt-get install autoconf automake groff
            sudo apt-get install curl lzop asciidoc
            sudo apt-get install u-boot-tools
        ```
    2. 安装 repo
        ```
            curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o repo
            chmod +x repo

            vim repo
        ```
            搜索出 “google” 关键字，把对应行改为链接
            ```shell
                REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'
            ```
            后修改 .bashrc 文件，把repo路径添加到环境变量中

    3. 创建一个本地 yocto 工程目录
        ``` 
            mkdir yocto_project && cd yocto_project
        ```
    4. repo init 初始化仓库
        ```shell
            repo init  -u  [github-url]
                example1: repo init -u https://github.com/STMicroelectronics/oe-manifest.git -b refs/tags/openstlinux-5.10-dunfell-mp1-21-03-31
                example2: repo init -u git://git.freescale.com/imx/fsl-arm-yocto-bsp.git -b imx-4.1-krogoth
        ```
    5. repo 仓库同步
        ```shell
            repo sync
        ```

    * 备注：以上两步对网络要求都比较高，因为有些仓库在国外，所以需要翻墙

    6. 设置环境变量
        ```
            DISTRO=<distro name> MACHINE=<machine name> source fsl-setup-release.sh -b <build dir>
                example1:DISTRO=fsl-imx-x11 MACHINE=imx6ulevk source fsl-setup-release.sh -b ./fsl_build_x11
                exampke2:DISTRO=openstlinux-weston MACHINE=stm32mp1 source layers/meta-st/scripts/envsetup.sh
        ```
        具体的 DISTRO 、 MACHINE 视不同Soc官方给定的手册而定,MACHINE 默认会成为主机的hostname, /etc/hostname
    7. 编译
        ```
            bitbake [image]
                example1:bitbake core-image-minimal
                example2:bitbake st-image-weston
        ```
        具体的 image 视不同Soc官方给定的手册而定

* yocto 编译后的目录架构
    1. 镜像生成目录路径
            `<yocto project>/<build dir>/tmp/deploy/images/imx6ulevk`
            or
            `<yocto project>/<build dir>/tmp/deploy/images/stm32mp1`

    2. 下载下来的uboot、kernel、根文件系统目录
        1. uboot 路径： 类似这种
            `<yocto project>/<build dir>/tmp/work/imx6ulevk-poky-linux-gnueabi/u-boot-imx/2016.03-r0/git`
            or
            `<yocto project>/<build dir>/tmp-glibc/work-share/stm32mp1/uboot-source`
        2. Linux 路径： 类似这种
            `<yocto project>/<build dir>/tmp/work-shared/imx6ulevk/kernel-source`
            or
            `<yocto project>/<build dir>/tmp-glibc/work-share/stm32mp1/kernel-source`
        3. 根文件系统目录
            `<yocto project>/<build dir>/tmp/deploy/images/imx6ulevk/core-image-minimal-imx6ulevk-          20190621012322.rootfs.tar.bz2`
            or
            `<yocto project>/<build dir>/tmp-glibc/work-share/stm32mp1/kernel-source`
        
        一般工作方法：
        把以上3件套拿出来单独管理

* layer 目录架构 && 文件说明
    1.  .bbappend && .bb 文件
        + 创建append文件时，必须使用对应recipe相同的名字。例如someapp_2.7.bbappend必须应用于someapp_2.7.bb，这意味着这两个文件是版本特定的。如果对应的recipe重命名升级了版本，你也需要同时改动.bbappend文件名。如果检测到.bbappend文件没有所匹配的recipe，BitBake会显示错误
        + 使用.bbappend文件，你可以在无需拷贝另一个Layer的recipe到你的Layer中，就能增加或修改内容。.bbappend文件在你的Layer中，而被附加内容的.bb文件则在另一个Layer中
    2.  <layer dir> 目录下的 conf/layer.conf
        + 最简单的方式是拷贝一份已有配置到你的Layer配置目录中，然后根据需要改动它。
        + demo
            ```
                # We have a conf and classes directory, add to BBPATH
                BBPATH .= ":${LAYERDIR}"

                # We have recipes-* directories, add to BBFILES
                BBFILES += "${LAYERDIR}/recipes-*/*/*.bb \
                            ${LAYERDIR}/recipes-*/*/*.bbappend"

                BBFILE_COLLECTIONS += "yoctobsp"
                BBFILE_PATTERN_yoctobsp = "^${LAYERDIR}/"
                BBFILE_PRIORITY_yoctobsp = "5"
                LAYERVERSION_yoctobsp = "4"
                LAYERSERIES_COMPAT_yoctobsp = "warrior"
            ```
        + 相关定义
            + BBPATH: 将此Layer根目录添加到BitaBake搜索路径中。利用BBPATH变量，BitBake可以定位类文件（.bbclass），配置文件，和被include的文件。BitBake使用匹配BBPATH名字的第一个文件，这与给二进制文件使用的PATH变量类似。同样也推荐你为你的Layer中类文件和配置文件起一个唯一的名字。
            + BBFILES: 定义Layer中recipe的路径
            + BBFILE_COLLECTIONS: 创建唯一标识符以给OE构建系统参照。此示例中，标识符"yoctobsp"代表"meta-yocto-bsp"Layer。
            + BBFILE_PATTERN: 解析时提供Layer目录
            + BBFILE_PRIORITY: OE构建系统在不同Layer找到相同名字recipe时所参考的使用优先级
            + LAYERVERSION: Layer的版本号。你可以通过LAYERDEPENDS变量指定使用特定版本号的Layer

* layer 操作
    * layer的新增和添加到bitbake的构建命令中
        1. 新增layer
            ```
                bitbake-layers create-layer your_layer_name
            ```
        2. 启用自建的layer,需要在 <build dir> 目录下的 conf/bblayers.conf 的 BBLAYERS 变量中，新增指定layer的路径 ,或使用命令 
            ```
                bitbake-layers add-layer your_layer_name
        ```

## 定制化镜像
* 定制化镜像
    1. 修改 `<build dir>/<conf>/local.conf` 文件，

    2. 使用自定义`IMAGE_FEATURES` 和 `EXTRA_IMAGE_FEATURES`定制化镜像

    3. 使用自定义.bb文件定制化镜像


## 编写 recipe 文件
* recipe 常规流程
    `establish the recipe` -> `fetch source files` -> `unpack source files` -> `patching source files` -> `add licensing information` -> `add configurations` -> `compilation` -> `autotools or cmake ?` -> `need supporting services` -> `packing` -> `provide post-installation scripts` -> `perform runtime testing`

* recipe文件创建
    1. devtool add: 帮助创建recipe和有助于开发的环境的命令
    2. recipetool create: Yocto Project提供，自动根据代码文件创建基本recipe的命令
    3. 已有recipe: 定位并改造一个功能和你需要的类似的recipe

* recipe文件
    1. 添加保存Recipe
        OE构建系统通过Layer的conf/layer.conf中BBFILES变量定位recipe。修改 BBFILES  变量来控制文件的添加
    2. .bb文件构建过程产生的中间文件路径
        demo:设备目标系统是qemux86-poky-linux。假定你的recipe名叫foo_1.3.0.bb，这种情况下，构建系统使用的工作目录是 `build/tmp/work/qemux86-poky-linux/foo/1.3.0-r0`

* recipe文件
    * 语法
        1. `SRC_URI`
            +  对于git仓库的，须指定`SRCREV` = "d6918c8832793b4205ed3bfede78c2f915c23385"
            + `SRC_URI[md5sum]` 和 `SRC_URI[sha256sum]`
            + demo
                ```
                    SRC_URI = "${DEBIAN_MIRROR}/main/a/apmd/apmd_3.2.2.orig.tar.gz;name=tarball \
                                    ${DEBIAN_MIRROR}/main/a/apmd/apmd_${PV}.diff.gz;name=patch"
                    or
                    SRC_URI = "git://github.com/STMicroelectronics/gcnano-binaries;protocol=https;branch=gcnano-6.4.3-binaries"
                ```
            + 若`SRC_URI`指向本地文件
                ```
                    SRC_URI = "http://dist.schmorp.de/rxvt-unicode/Attic/rxvt-unicode-${PV}.tar.bz2 \
                                file://xwc.patch \
                                file://rxvt.desktop \
                                file://rxvt.png"
                ```
                当使用file://指定本地文件时，构建系统从本地获取文件。这个路径相对于FILESPATH变量值，根据特定顺序寻找文件：${BP}, ${BPN}, 和 files。目录默认时recipe或append文件所在目录的子目录。
        2. `PV` 版本号 or 上一个版本+当前版本
        3. `DEPENDS` DEPENDS指定的项应该是其他recipe的名字，指定构建时依赖
        4. `RDEPENDS` 指定基于包的运行时依赖
    * 使用Headers与设备连接
        1. recipe构建一个需要和设备通讯，或是需要API访问自定义kernel的应用，你需要提供适当的头文件`xxx.inc`
        2. 不应当改动已有的 `meta/recipes-kernel/linux-libc-headers/linux-libc-headers.inc`文件。这些头文件用来构建libc，不能被自定义或者设备特定的头文件信息改动。如果你通过修改头文件的方式改动了libc，所有其他用到libc的应用程序都会收到影响
            + 不要改动 linux-libc-headers.inc. 这个文件是libc系统的一部分，不是你用来直接访问kernel的东西，你需要通过libc调用访问libc
            + 需要直接访问设备的应用，需要提供必要的头文件，或者建立起基于特定设备头文件包的依赖
    * 编译
        1. 错误的普遍原因
            + 并行构建失败
            + 错误使用主机路径
            + 无法找到需要的库/头文件。 如果构建时依赖因为没有定义在DEPENDS中，或是构建过程使用的查询路径不正确，配置过程也没有检测到这点而导致构建时依赖缺失，编译过程会失败
    * 安装
    * 启用系统服务
    * 打包
    * Recipe间共享文件

* 3.10 构建

* 3.34 Using Wayland and Weston

* 3.4


## bitbake 使用
* bitbake
    1.  查找包的原路径：
            bitbake -e hello | grep ^SRC_URI
        查找包的安装路径：
            bitbake -e hello | grep ^S=
        查找内核的路径
            bitbake -e linux | grep ^S=
        查看hello这个package能否被bitbake：
            bitbake -s | grep hello

    2. 单独编译
        bitbake命令单独编译u-boot：
            bitbake -c compile -f u-boot-imx
            bitbake -c deploy -f u-boot-imx        //部署编译生成的u-boot镜像到deploy
        bitbake命令单独编译kernel：
            bitbake -c compile -f linux-imx       //编译内核
            bitbake -c deploy -f linux-imx        //部署内核镜像到deploy目录
