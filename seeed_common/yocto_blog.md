# Yocto开发讲解系列


## blog 详解目录
* [Yocto开发篇](https://blog.csdn.net/fulinus/category_10445379.html)


## yocto 基础
* 构建项目的流程
    1. Fetch
    2. Extract
    3. Patch
    4. do_prepare_recipe_sysroot
        将 recipe 依赖的一些文件放到 sysroot 中。 此任务在${WORKDIR}目录中创建两个sysroot(recipe-sysroot、recipe-sysroot-native) 子目录，以便在打包阶段，将recipe所依赖的其他recipe在 do_prepare_recipe_sysroot 任务时放到 sysroot 目录。两个 sysroot 子目录分别装目标 machine 和本地机器（编译机，host,ubuntu）使用到的执行程序和库等二进制文件。
    5. Configure
        类似 ./configure --prefiex=xx ,执行软件配置
    6. Build / do compile
        configure 之后，编译源代码，编译发生在B变量指向的目录中。缺省值情况下，B与S指向相同的目录。
        如果是线程的可执行程序 或 库文件，configure 和 build,可以在 bb 文件中写成空函数
        ```
            do_configure_prepend () {
            }
            do_configure () {
            }

            do_compile () {
            }
        ```
    7. Install
        install任务会从B指向的目录中复制文件到D指向的目录中。稍后会使用此目录中的文件进行打包
        查看D目录
        ```
            bitbake -e xxxx | grep ^D=
        ```
    8. Package
        会把编译输出文件的包splits,比如 debug 版本一个目录，release 版本一个目录
        + do_package

        + do_packagedata
            分析创建包的元数据，以便构建系统可以生成最终的包
        + d0_package_write_*
            根据包的类型（rpm,deb,ipk）创建实际的 packages，并放到 ${TMPDIR}/deploy目录下

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
            yocto_find *.bb *.bbappend *.inc *.conf *.bbclass| grep -v "build-openstlinuxweston-stm32mp1" | xargs grep -in KERNEL_EXTRA_FEATURES
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

poky]$ mylocate *.bb *.bbappend *.inc *.conf *.bbclass | grep -v "poky/build" | xargs grep -in KERNEL_EXTRA_FEATURES
/home/peeta/poky/meta/recipes-kernel/linux/linux-yocto-dev.bb:47:KERNEL_EXTRA_FEATURES ?= "features/netfilter/netfilter.scc features/taskstats/taskstats.scc"
/home/peeta/poky/meta/recipes-kernel/linux/linux-yocto-dev.bb:48:KERNEL_FEATURES_append = " ${KERNEL_EXTRA_FEATURES}"
/home/peeta/poky/meta/recipes-kernel/linux/linux-yocto-rt_5.4.bb:39:KERNEL_EXTRA_FEATURES ?= "features/netfilter/netfilter.scc features/taskstats/taskstats.scc"
/home/peeta/poky/meta/recipes-kernel/linux/linux-yocto-rt_5.4.bb:40:KERNEL_FEATURES_append = " ${KERNEL_EXTRA_FEATURES}"
/home/peeta/poky/meta/recipes-kernel/linux/linux-yocto-rt_5.8.bb:39:KERNEL_EXTRA_FEATURES ?= "features/netfilter/netfilter.scc features/taskstats/taskstats.scc"
/home/peeta/poky/meta/recipes-kernel/linux/linux-yocto-rt_5.8.bb:40:KERNEL_FEATURES_append = " ${KERNEL_EXTRA_FEATURES}"
/home/peeta/poky/meta/recipes-kernel/linux/linux-yocto_5.4.bb:48:KERNEL_EXTRA_FEATURES ?= "features/netfilter/netfilter.scc"
/home/peeta/poky/meta/recipes-kernel/linux/linux-yocto_5.4.bb:49:KERNEL_FEATURES_append = " ${KERNEL_EXTRA_FEATURES}"
/home/peeta/poky/meta/recipes-kernel/linux/linux-yocto_5.8.bb:49:KERNEL_EXTRA_FEATURES ?= "features/netfilter/netfilter.scc"
/home/peeta/poky/meta/recipes-kernel/linux/linux-yocto_5.8.bb:50:KERNEL_FEATURES_append = " ${KERNEL_EXTRA_FEATURES}"


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
* 变量定义
    1. SRC_URI
        指向源文件，每个recipes必须有一个指向源的 SRC_URI
        * 注： 
            1. 本地源文件在 SRC_URI 变量中的表示形式是 "file://"开头，加上路径
            2. 该变量句子格式中，被处理的补丁文件后缀可为： *.patch , *.diff
    2. DL_DIR
        指定源文件下载到响应的目录中。
        假如是多用户使用，可以指定一个yocto工程目录以外的文件夹，like: DL_DIR ?= "~/downloads".
        DL_DIR 路径下会有很多下载完成的软件包，对应软件包名 + ".done",表示已下载完成， xxx.done 文件只是一个标记文件，实则有效还是原压缩包，用 `ls -l`可以看到有对应文件的大小。
    3. SRCREV
        如果是通过git下载的话,该变量用来指定的版本构建
    4. PACKAGE_CLASSES
        在 build/conf/local.conf文件中，指定OpenEmbedded构建系统在打包数据时使用何种包管理。可以一种，可以多种
        like: PACKAGE_CLASSES ?= "package_rpm package_deb package_ipk package_tar"
    5. DEPLOY_DIR
        定义生成Image,SDK输出路径。like:build/tmp/deploy
    6. DEPLOY_DIR_*
        DEPLOY_DIR_RPM，DEPLOY_DIR_IPK 该系列变量，定义了不同包，例：rpm、ipk、deb类型包的放置路径。
    7. PACKAGE_ARCH
        定义子目录名，like: build/tmp/deploy/{PACKAGE_ARCH}
        ```
            bitbake -e learnyocto | grep ^PACKAGE_ARCH
            PACKAGE_ARCH="core2-64"
        ```
    8. TMPDIR
        OpenEmbedded 构建所有工作的及目录。 like:build/tmp
    9. TARGET_OS
        目标设备的操作系统。like: TARGET_OS="linux"
    10. PN
        用于构建包的recipe的名称，这个变量多种含义。当在输入文件的上下文中使用时，PN表示recipe的名称。like: recipes-kernel 的 ${PN}=kernel
        ```sh
            bitbake -e helloworld-m | grep ^PN
            PN = "hello-m"
        ```
    11. WORKDIR
        用来构建 recipe 的位置
        ```sh
            bitbake -e helloworld-m | grep ^WORKDIR=
            WORKDIR="/home/cxn/yocto/build/tmp/work/xxxx"
        ```
    12. PV
        用于构建包 recipe版本
        ```sh
            bitbake -e learnyocto | grep ^PV=
            PV="1.0+git..."
        ```
    13. PR
        构建包的 recipe 的修订版本
        ```sh
            bitbake -e learnyocto | grep ^PR=
            PR="r0"
        ```
    14. S
        每个recipe在build/目录中都有一个子目录，解压后的源代码就在那里。"S"变量指向该子目录中的recipe源代码或源文件。 注：不要在S指定的路径下修改代码，无效
        like:
        ```sh
            bitbake -e learnyocto | grep ^S
            S="xxx/xx/build/tmp/work/core2-64-poky-linux/learnyocto/1.0+gitAUTOINC+cac0ee57e2f-r0/git"
            # cac0ee57e2f 为git的commit-id前面的一部分
        ```
        learnyocto的路径可见 PACKAGE_ARCH(core2-64) 控制了该项目放在 build/tmp/work/core2-64-poky-linux 目录下，而不是build/tmp/work/qemux86_64-poky-linux 目录下
    15. BPN
        用于构建包的recipe名称，BPN变量是PN的变种，通常删除了前缀和后缀
        ```sh
            bitbake -e learnyocto | grep ^BPN=
            BPN="learnyocto"
        ```
    16. FILESPATH
        + 构建系统 `搜索` 补丁 和 文件 时使用的默认目录集合。
        + 在构建过程中，当 recipe 的SRC_URI 语句中查找由每个 file://URI 指定的文件和补丁时，bitbake 会按照指定顺序在 FILESPATH 搜索每个目录。
        + FILESPATH 变量默认值是在 base.bbclass中定义，可在 meta/class 目录下找到相关定义
            ```sh
                FILESPATH = "${@base_set_filespath(["${FILE_DIRNAME}/${BP}", \
                    "${FILE_DIRNAME}/${BPN}", "${FILE_DIRNAME}/files"], d)}"
            ```
        + 使用查看的命令
            ```sh
                # 目录架构(注意目录文件夹的名字)
                    /meta/recipes-extended/iputils/iputils/xxxx.patch


                # .bb 文件
                SRC_URI = "git://github.com/iputils/iputils \
                file://0001-ninfod-change-variable-name-to-avoid-colliding-with-.patch \
                file://0001-ninfod-fix-systemd-Documentation-url-error.patch \
                file://0001-rarpd-rdisc-Drop-PrivateUsers.patch \
                file://0001-iputils-Initialize-libgcrypt.patch \

                # shell
                bitbake -e iputils | grep ^FILESPATH
                FILESPATH="/home/peeta/poky/meta/recipes-extended/iputils/iputils-s20190709/poky:/home/peeta/poky/meta/recipes-extended/iputils/iputils/poky:/home/peeta/poky/meta/recipes-extended/iputils/files/poky:/home/peeta/poky/meta/recipes-extended/iputils/iputils-s20190709/qemux86-64:/home/peeta/poky/meta/recipes-extended/iputils/iputils/qemux86-64:/home/peeta/poky/meta/recipes-extended/iputils/files/qemux86-64:/home/peeta/poky/meta/recipes-extended/iputils/iputils-s20190709/qemuall:/home/peeta/poky/meta/recipes-extended/iputils/iputils/qemuall:/home/peeta/poky/meta/recipes-extended/iputils/files/qemuall:/home/peeta/poky/meta/recipes-extended/iputils/iputils-s20190709/x86-64:/home/peeta/poky/meta/recipes-extended/iputils/iputils/x86-64:/home/peeta/poky/meta/recipes-extended/iputils/files/x86-64:/home/peeta/poky/meta/recipes-extended/iputils/iputils-s20190709/:/home/peeta/poky/meta/recipes-extended/iputils/iputils/:/home/peeta/poky/meta/recipes-extended/iputils/files/"
            ```
    17. EXTRA_OECONF
        针对 `autotools` 项目， 通过 EXTRA_OECONF 或 PACKAGECONFIG_CONFARGS 变量传达 configuration 选项参数
        ```demo
            EXTRA_OECONF = "--with-platform=${GRUBPLATFORM} \
                --disable-grub-mkfont \
                --program-prefix="" \
                --enable-liblzma=no \
                --enable-libzfs=no \
                --enable-largefile "
        ```
    18. EXTRA_OECMAKE
        针对 `cmake` 项目，configuration 选项可以通过使用 EXTRA_OECMAKE 变量传达。
        ```
            EXTRA_OECMAKE = "-DBUILD_SHARED_LIBS=ON \
                 -DCMAKE_DISABLE_FIND_PACKAGE_Boost=TRUE \
                 -DHAVE_BOOST_BYTESWAP=FALSE \
                 -DCMAKE_CXX_STANDARD=11 \
                 -DCMAKE_CXX_STANDARD_REQUIRED=OFF \
                 -DLIB_SUFFIX=${@d.getVar('baselib').replace('lib', '')}"
        ```
    19. EXTRA_OEMAKE
        针对 `Makefile` 项目，configuration 选项可以通过使用 EXTRA_OEMAKE 变量传达。
        ```
            EXTRA_OEMAKE = "MKDIR='mkdir' CP='cp -f' RM='rm' LDFLAGS='-lpthread -ldl' OE_BUILD=1"
        ```
    20. CFLAGS 或 CXXFLAGS
        gcc 、g++编译选项
        ```
            CFLAGS += "-I${S}/include"
            CFLAGS_append = " -fPIC"
            CXXFLAGS_append = " -fPIC"
        ```
    21. PKGD
        在将包拆分为单个包之前，包的目标目录
    22. PKGDESTWORK
        do_package任务用来保存包元数据的临时工作目录
    23. PKGDEST
        拆分后包的父目录
    24. PKGDATA_DIR
        一个共享的全局状态目录，用于保存在打包过程中生成的打包元数据（packaging metadata）。打包过程从 PKGDESTWORK 变量指定的目录中赋值元数据到 PKGDATA_DIR 目录
        ```sh
            bitbake -e xxxx | grep ^PKGDATA_DIR
        ```
    25. STAGING_DIR_HOST
        recipe 依赖的组件存放的sysroot路径，比如：recipe-sysroot。包含有一些 xxx 项目依赖的库和头文件，该目录下的组件，尤其库文件，后期会整合到目标设备的文件系统中。 这些库的编译架构决定了只能在目标机器行运行
    26. STAGING_DIR_NATIVE
        在build响应的recipe时，需要被用到一些组件工具等，例如，cmake、gcc等，后期不会整合到目标设备的文件系统中。
        该目录先的 cmake 和 gcc 是可以运行的。
    27. STAGING_DIR_TARGET
        与 STAGING_DIR_HOST 变量类似， recipe 依赖的组件存放的sysroot路径
    28. FILES
        用来定义哪些文件会在打包阶段被使用，也就是一些 recipe 构建完生成的执行文件、库、头文件或者其他一些文件
        ```sh
            bitbake -e learnyocto | grep ^FILES
            FILES=""
            ...
            FILES_SOLIBSDEV="/lib/lib*.so /usr/lib/lib*.so"
            FILES_learnyocto="/usr/bin/* /usr/sbin/* /usr/libexec/* /usr/lib/lib*.so.*             /etc /com /var             /bin/* /sbin/*             /lib/*.so.*             /lib/udev /usr/lib/udev             /lib/udev /usr/lib/udev             /usr/share/learnyocto /usr/lib/learnyocto/*             /usr/share/pixmaps /usr/share/applications             /usr/share/idl /usr/share/omf /usr/share/sounds             /usr/lib/bonobo/servers"
            FILES_learnyocto-bin="/usr/bin/* /usr/sbin/*"
            FILES_learnyocto-dbg="/usr/lib/debug /usr/lib/debug-static /usr/src/debug"
            FILES_learnyocto-dev="/usr/include /lib/lib*.so /usr/lib/lib*.so /usr/lib/*.la                 /usr/lib/*.o /usr/lib/pkgconfig /usr/share/pkgconfig                 /usr/share/aclocal /lib/*.o                 /usr/lib/learnyocto/*.la /lib/*.la                 /usr/lib/cmake /usr/share/cmake"
            FILES_learnyocto-doc="/usr/share/doc /usr/share/man /usr/share/info /usr/share/gtk-doc             /usr/share/gnome/help"
            FILES_learnyocto-locale="/usr/share/locale"
            FILES_learnyocto-src=""
            FILES_learnyocto-staticdev="/usr/lib/*.a /lib/*.a /usr/lib/learnyocto/*.a"
        ```
    29. IMAGE_INSTALL
        列出要从"包源"区域安装的包的基本集
    30. PACKAGE_EXCLUDE
        用来指定不应安装到镜像（image）中的包
    31. IMAGE_FEATURES
        指定要包含在镜像中的特性，这些特性中的大多数映射到附加的安装包
        ```
            bitbake -e core-image-sato | grep ^IMAGE_FEATURES=
            IMAGE_FEATURES="debug-tweaks hwcodecs package-management splash ssh-server-dropbear x11-base x11-sato"
        ```
    32. PACKAGE_CLASSES
        用来指定是什么类型的包（rpm、deb、ipk）,帮助将包放在何处
    33. IMAGE_LINGUAS
        确定为其安装其他语言支持包的语言
        指定在根文件系统构造过程中要安装到镜像中，比如，添加中文语言支持，可在 build/conf/local.conf  中添加`IMAGE_LINGUAS_append= " zh-cn"`,加完之后，需要重新source xxx-env 环境脚本
    34. PACKAGE_INSTALL
        传递给包管理器一安装到镜像中的包的`最终列表`
    35. IMAGE_ROOTFS
        指向文件系统创建过程中的目标位置
        ```
            bitbake -e core-image-sato | grep ^IMAGE_ROOTFS=
            IMAGE_ROOTFS="/home/peeta/poky/build/tmp/work/qemux86_64-poky-linux/core-image-sato/1.0-r0/rootfs"
        ```
    36. IMAGE_MANIFEST
        文件清单（xxx.manifest）与 根文件系统镜像位于同一个目录中。此文件逐行列出已安装的软件包。有用
    37. IMAGE_FDTYPES
        指定文件系统类型和压缩类型。通常ext4的文件系统镜像较大，因为文件系统有预留空间。
        ```sh
            bitbake -e core-image-sato | grep ^IMAGE_FSTYPES=
            IMAGE_FSTYPES=" tar.bz2 ext4"
        ```

    38. SDK_EXT_TYPE
        控制是否将共享状态工件复制到可扩展SDK中
    39. SDK_INCLUDE_TOOLCHAIN
        指定构建可扩展SDK时，是否包含工具链
        ```sh
            # 查看SDK中的变量
             [build]$ bitbake -e core-image-sato | grep "^TOOLCHAIN_HOST_TASK=\|^SDKMACHINE=\|^SDKIMAGE_FEATURES=\|^TOOLCHAIN_TARGET_TASK=\|^SDKPATH=\|^SDK_HOST_MANIFEST=\|^SDK_TARGET_MANIFEST="
                SDKIMAGE_FEATURES="dev-pkgs dbg-pkgs src-pkgs "
                SDKMACHINE="x86_64"
                SDKPATH="/opt/poky/3.1.2"
                SDK_HOST_MANIFEST="/home/peeta/poky/build/tmp/work/qemux86_64-poky-linux/core-image-sato/1.0-r0/x86_64-deploy-core-image-sato-populate-sdk/poky-glibc-x86_64-core-image-sato-core2-64-qemux86-64-toolchain-3.1.2.host.manifest"
                SDK_TARGET_MANIFEST="/home/peeta/poky/build/tmp/work/qemux86_64-poky-linux/core-image-sato/1.0-r0/x86_64-deploy-core-image-sato-populate-sdk/poky-glibc-x86_64-core-image-sato-core2-64-qemux86-64-toolchain-3.1.2.target.manifest"
                TOOLCHAIN_HOST_TASK="nativesdk-packagegroup-sdk-host packagegroup-cross-canadian-qemux86-64 nativesdk-intltool nativesdk-glib-2.0"
                TOOLCHAIN_TARGET_TASK="packagegroup-core-standalone-sdk-target target-sdk-provides-dummy     packagegroup-core-boot     packagegroup-base-extended               learnyocto run-postinsts rpm dnf psplash packagegroup-core-ssh-dropbear packagegroup-core-x11-base packagegroup-core-x11-sato"

        ```

    40. DISTRO
        表示发行版的简称。 一般在 meta-xxx/conf/distro/xxx.conf 文件中
    41. DISTRO_VERSION
        发行版本
    42. VARIANT
        版本的变种
        VARIANT="perf"
        #or
        VARIANT="debug"
    43. PACKAGE_DEBUG_SPLIT_STYLE
        决定在创建与gdb调试器一起使用的*.dbg包时，如何分割二进制文件和调试信息
    44. SERIAL_CONSOLE && SERIAL_CONSOLES
        指定控制台的波特率
        ```sh
            SERIAL_CONSOLE = "115200 ttyS0"
            SERIAL_CONSOLES = "115200;ttyS0 115200;ttyS1"
        ```
    45. FULL_OPTIMIZATION
        编译优化系统时传入TARGET_CFLAGS 和 CFLAGS 的选项中，默认是 `-O2 -pipe ${DEBUG_FLAGS}`
    46. ENABLE_BINARY_LOCALE_GENERATION
        控制在构建期间为glibc生成那些 locals (区域)，对于一些RAM不超过64M的设备，有用
    47. USE_LDCONFIG
        控制是否安装ldconfig，like：不安装  USE_LDCONFIG = "0"
    48. PREFERRED_VERSION
        优先使用某个版本，前提是你的recipe支持多个版本，并且是这些版本中的一个才可以用。以 PN(recipe)做后缀，赋值为版本号，可以用通配符 "%"
        ```
            PREFERRED_VERSION_autoconf = "2.68"
            PREFERRED_VERSION_readline = "5.2"
            PREFERRED_VERSION_python = "3.4.0"
            PREFERRED_VERSION_linux-yocto = "4.12%"
        ```
    49. USE_DEVFS
        确定devtmpfs是否用于/dev入口，一般默认值为"1"
    50. DEPLOY_DIR_IMAGE
        指向构建系统 用于放置准备部署到目标机器上的镜像和其他相关输出文件的目录。该目录是特定于机器的，因为他包含 MACHINE 名，默认情况下是 ${DEPLOY_DIR}/images/${MACHINE}/中
    51. PACKAGE_ARCH
        生成一个或多个体系结构的软件包


    52. inherit
        继承一个或者多个class类。通常是继承 bbclass 文件的类
    53. require
        引用共性 .inc 文件
    54. DEPENDS
        告知当前的recipe依赖其他的recipe
        ```sh
            DEPENDS = "bar"
            DEPENDS_append = "baz"
            或者这样：
            DEPENDS = "bar"
            DEPENDS += "baz"
            或者这样：
            DEPENDS = "bar baz"
        ```
    56. THISDIR
        当前bb文件所在的当前路径
    57. FILESEXTRAPATHS
        用来扩展构建系统在处理recipe和append文件时，查找文件和补丁时使用的搜索路径。bitbake在处理recipes时默认使用的目录最初由 FILESPATH 变量定义，可以通过 FILESEXTRAPATHS 变量来扩展所搜路径

    58. CORE_IMAGE_EXTRA_INSTALL
        允许您基于核心映像类向映像添加额外的包。
    59. BB_NUMBER_THREADS = "32" 
        修改BitBake可以使用的线程，放在 build/conf/local.conf
    60. PARALLEL_MAKE = "-j 32" 
        修改make可以使用的线程,放在 build/conf/local.conf
        



* 镜像文件制作
    * 详细的流程图可以参考： [Bitbake 镜像文件制作](https://fulinux.blog.csdn.net/article/details/110400636)
    * do_rootfs任务
        为映像创建根文件系统（文件和目录结构）


## build目录架构
* tmp
    * work目录
        work
          |- ${PACKAGE_ARCH}-poky-${TARGET_OS}
            |- ${PN}
              |- ${PV}-${PR}        ...... WORKDIR
                |- {BPN}-${PV}      ...... S / B    # bitbake -e xxx|grep ^B=
                |- image            ...... D
                |- package          ...... PKGD
                |- pkgdata          ...... PKGDESTWORK
                |- package-split    ...... PKGDEST
                    |-${PN}         ...... 
                |- recipe-sysroot
                |- recipe-sysroot-native
          |- ${PACKAGE_ARCH}-poky-${TARGET_OS}
            |- ${PN}
              |- ${PV}-${PR}        ...... WORKDIR
                |- {BPN}-${PV}      ...... S / B
                  |- image          ...... D

## ToolChain 或交叉编译器
* 介绍如何生成toolchain SDK [应用开发的SDK或toolchain或gcc](https://fulinux.blog.csdn.net/article/details/110494600)
    + 编译标准SDK
        1. shell
            ```
                bitbake core-image-sato -c populate_sdk
            ```
            需要一段时间，输出文件路径
            ```sh
                ls tmp/deploy/sdk/
                    poky-glibc-x86_64-core-image-sato-core2-64-qemux86-64-toolchain-3.1.2.host.manifest
                    poky-glibc-x86_64-core-image-sato-core2-64-qemux86-64-toolchain-3.1.2.sh
                    poky-glibc-x86_64-core-image-sato-core2-64-qemux86-64-toolchain-3.1.2.target.manifest
                    poky-glibc-x86_64-core-image-sato-core2-64-qemux86-64-toolchain-3.1.2.testdata.json
            ```
    + 安装SDK
        直接上操作，默认是安装到 /opt 路径下，也可以指定
        ```sh
            build]$ ./tmp/deploy/sdk/poky-glibc-x86_64-core-image-sato-core2-64-qemux86-64-toolchain-3.1.2.sh #安装 该脚本应该可以指定安装目录
            $ . /opt/poky/3.1.2/environment-setup-core2-64-poky-linux   # 每次启用 SDK环境 。 类似于yocto工程前的source xxx.env
            $ ls /opt/poky/3.1.2/
            $ $CC -v    # 查看编译器的版本，名称信息
        ```
