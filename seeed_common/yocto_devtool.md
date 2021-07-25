# Yocto-devtool


## devtool 子命令
* devtool 有许多子命令
    1. devtool add
        自动创建一个 recipe
        + demo1：添加本地的源文件
            ```sh
                $ ls -a ~/code/helloyocto/
                    main.c Makefile
                [build] $ devtool add helloyocto ~/code/helloyocto #add的使用 yocto工程外部区域的目录
                [build] $ ls workspace/
                [build] $ cat workspace/recipes/helloyocto/helloyocto.bb
            ```
            该命令会创建 workspace 目录（若一开始不存在）
        + demo2: 添加github或gieee上的源码
            ```
                [build] $ devtool add --srcbranch develop learnyocto [git-source-url：https://xxxx.git]
                [build] $ ls workspace/source/learnyocto
            ```
    2. devtool modify
        将指定现有的recipe来确定从何处获取源代码以及如何对其进行修补
    3. devtool upgrade
        更新现有的recipe
    4. devtool build
        编译 recipe。 类似于bitbake。使用devtool build 命令时，必须使用recipe的跟名称（以上为 helloyocto ）
        ```
            [build] $ devtool build helloyocto
            [build] $ ls workspace/sources/helloyocto/oe-workdir/image/usr/bin/helloyocto
        ```
    5. devtool reset
        从workspace目录中删除指定的recipe。
    6. devtool status
        查看workspace中有哪些项目，以及各自的源码路径
    7. devtool build-image
        构建image,并将 workspace 中的recipes包含到 image 文件系统中。适用于开发和修改验证的中途使用，如果成功了，就集合到某个metadata layer目录中。
    8. devtool finish
        将在 workspace 目录中开发的 recipe 集成到正式的 meta-layer中去
        ```
            devtool finish learnyocto meta-mylayer
        ```
        假如还要将可执行文件放到生成的文件系统中去，还得修改对应的.bb文件，修改 `IMAGE_INSTALL += xxx` 来添加，添加完后还要 source xxx_env 
        ```sh
            bitbake core-image-sato -c cleanall
            bitbake core-image-sato

            # 确认是否添加成功
            build/tmp-qemux86-64/work/qemux86_64-poky-linux/core-image-sato/1.0-r0/rootfs/usr/bin/learnyocto 
        ```
        查看安装软件清单也可以确认
        ```sh
            cat build/tmp/deploy/images/xxxx/xxxxx.manifest
        ```
    9. devtool deploy-target
        将 recipe的构建的 do_install 任务中安装的所有文件直接输出部署到`运行着的目标机器`上，目标机器上面需要运行ssh服务。
            开发机器            目标机器
                |                   |
                |                   |
                | ----------------> |
        ```sh
            [build] $  devtool deploy-target learnyocto  root@192.168.1.101:/   # 该目标不会自动部署程序所依赖的库和程序
            [目标设备] $ ./usr/bin/learnyocto
        ```

        + 部署失败原因
            + 需要部署的程序正在运行，或者库被依赖使用中
            + 目标机器上没有安装应用程序所依赖的库或者其他程序。
    10. devtool undeploy-target
        与上述 devtool deploy-target 命令相似，将我们前面部署到正在机器上的可执行程序卸载，移除。
        ```sh
            devtool undeploy-target learnyocto root@192.168.1.101:/ # 如果部署了多个应用程序，可以使用 -a 选项将他们全部删除，从而将目标设备恢复到其原始状态
        ```
    11. 
        
 
* 命令帮助说明
    1. 
        ```sh
            devtool -h
            devtool add --help
        ```

## devtool 的工作目录架构
* devtool 使用一个 workspace层来完成构建。是整个工具使用的公共工作区。
    + 目录架构
        build
            |- workspace-layer
                |- attic
                |-appends
                |-conf
                |-recipes
                |-sources




