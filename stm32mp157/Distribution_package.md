# Distribution Package

## 操作流程（这些步骤因为设计到国外一些网站，建议开发环境可以翻墙）
* 开发环境 翻墙 操作
    1. seeed 服务器上
        1. 登录上服务器之后，执行如下命令(导出环境变量),即可访问国外的服务器
            ```shell
                export ALL_PROXY='socks5://192.168.5.153:1081/'
                export all_proxy='socks5://192.168.5.153:1081/'
            ```
        2. 验证
            ```
                curl www.google.com
            ```
    2. 本地ubuntu服务器
        1. 安装一个 clash，速鹰
            [相关介绍](https://suying999.net/user/tutorial?os=linux&client=clash)
        2. 进入clash可执行文件的文件夹路径，下载订阅配置文件。下面这个链接应该不行的，自己购买。
            ```shell
                wget -O config.yml https://dingyue.suying666.info/link/BJGJoKsNaofBesKs?clash=1&log-level=info
            ```
            即 clash 和 config.yaml 文件同一目录
        3. 执行
            ```shell
                ./clash -f config.yaml
            ```
        4. 点击速鹰网页的第4步的 `Clash Dashboard` 的链接，选择代理点。
            该步骤下需注意：跳转到 `Clash Dashboard` 的外链接网页后，在相应网页的左边有 `设置` 一栏，点击，在 `代理模式` 中选择 `全局`，不然终端程序不能访问外网。
        5. 设置 ubuntu 的网络设置，代理设置为手动
            当不用外网是，代理选择 禁用 即可。
        6. 验证
            ```shell
                curl www.google.com
            ```


* 开发操作
    1. 创建一个存放 distribution_package 的目录，example: distribution_package
    2. 在 distribution_package 文件夹里，执行操作
        ```shell
            repo init -u https://github.com/STMicroelectronics/oe-manifest.git -b refs/tags/openstlinux-5.10-dunfell-mp1-21-03-31
        ```
        * 备注：因为 repo 拉取某些库会涉及到国外一些网站，故 repo 需要处理，具体参考 [清华源repo](https://mirrors.tuna.tsinghua.edu.cn/help/git-repo/)
    3. init完成之后，执行如下命令 sync
        ```shell
            repo sync
        ```
        * 备注：该命令执行前，最好全局翻墙
    4. 环境初始化
        ```shell
            DISTRO=openstlinux-weston MACHINE=stm32mp1 source layers/meta-st/scripts/envsetup.sh
        ```
    5. 构建工程
        ```shell
            bitbake st-image-weston
        ```

## yocto && bitbake 工程
* 工程生成的固件路径
    1. 镜像文件路径
        ```
            build-<distro>-<machine>/tmp-glibc/deploy/images/stm32mp1
        ```


## 疑问整理
1. yocto怎么单独编译 kernel,uboot,rootfs 和 TF-A
    * 以 编译kernel为例
        ```shell
            bitbake -s |grep linux   #查看 linux 关键字，以便定位 kernel,例如 stm32mp157c 类型，就可以定位到 linux-stm32mp
            bitbake -e linux-stm32mp | grep ^S= #定位内核源码的安装包，方便自己查看
            bitbake -c cleansstate linux-stm32mp
            bitbake  -c menuconfig linux-stm32mp    #配置内核
            bitbake linux-yocto -c diffconfig   #查看修改后的内核配置项
            bitbake -c compile -f linux-stm32mp #编译kernel
            bitbake -c deploy -f linux-stm32mp  #部署镜像到 deploy 目录
        ```
2. yocto 怎么编译文件系统
3. yocto 怎么编写内核驱动代码，怎么生成.ko文件
4. yocto 怎么编译工具链


## 踩坑
* 这些坑是从 mac ubuntu18.04上面踩的，在 seeed ubuntu18.04上暂无出现
    1. 增大磁盘的空间
        1. 处理 在虚拟机中把磁盘增大后，他不会把原磁盘直接增大，而是增大剩余空间，需要把剩余空间添加分区，后再挂载到文件系统中
        [在Linux下对未分配剩余空间分区挂载](https://blog.csdn.net/chiyanxi1706/article/details/100799682?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.control&dist_request_id=1328603.58688.16151957706854447&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.control)

    2. 增大swap区的大小
        1. ubuntu系统线增大交换区大小的操作[链接](https://blog.csdn.net/m0_46537958/article/details/108469587)

    3. 增大 RAM disk的大小
        1. 调整ubuntu RAM disk 的大小。（目前没尝试）[Ubuntu如何设置和调整Ram Disk大小](https://xiaohost.com/535.html)
        2. 在 yocto project/build/conf/local.conf 的最后加上 ` XZ_MEMLIMIT = "70%" ` 的语句，来增大编译时使用RAM的上限