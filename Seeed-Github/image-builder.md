# image-builder

* 操作方式
    ```sh
        demo: 生成 stm32mp157 img 镜像文件的操作
        ./publish/seeed-stm32mp1-stable.sh
    ```

* 相关说明
    1. `publish/seeed-stm32mp1-stable.sh`
        - 需添加代理，本公司的 cache
            ```sh
                export apt_proxy=192.168.4.40:3142/ #具体问宝哥，有可能有变动

                # 相关链接
                http://192.168.4.40:5000/
            ```
        - 是否上传到本公司的服务器，自行决定
            ```sh
                ssh_svr=192.168.1.78
                ssh_user="cxn@${ssh_svr}"
                server_dir="/home/public/share/stm32mp135"

                #upload:
                ssh ${ssh_user} mkdir -p ${server_dir}
                echo "debug: [rsync -e ssh -av ./${full_name}.img.xz ${ssh_user}:${server_dir}/]"
                rsync -e ssh -av ./${full_name}.bmap ${ssh_user}:${server_dir}/ || true
                rsync -e ssh -av ./${full_name}.img.xz ${ssh_user}:${server_dir}/ || true
                if [ -f "${full_name}.img.xz.job.txt" ]; then
                    rsync -e ssh -av ./${full_name}.img.xz.job.txt ${ssh_user}:${server_dir}/ || true
                fi
                rsync -e ssh -av ./${full_name}.img.xz.sha256sum ${ssh_user}:${server_dir}/ || true
                ssh ${ssh_user} "bash -c \"chmod a+r ${server_dir}/${full_name}.*\""
            ```
        - 配置要根据具体修改
            ```sh
                options="--img-2gb ${target_name}-${image_name} --dtb stm32mp135 --force-device-tree stm32mp135f-dk.dtb --enable-uboot-cape-overlays"
            ```
    2. `configs/xxxx.conf`
        根据自己需求，添加相关平台的.conf文件,配置kernel镜像的
        - deb仓库
            ```sh
                repo_external_server="" # 指向自己存放kernel.deb的仓库，可参考 aptly 文档
                repo_external_key="pgp-key.public" # 修改为自己发布deb时用的private-key对应的publish-key，并把 publish-key 放到 target/kering/ 路径下

                repo_external_pkg_list="linux-image-5.10.61-stm32-r1"   # kernel的名字
            ```
    3. `tools/hwpack/xxx.conf`
        根据自己需求，添加相关平台的.conf文件,配置uboot bin的
        - ng
            ```sh
                conf_bl_listfile="bootloader-ng"    # 该 bootloader-ng 文件要修改，修改完后放到 对应的http链接，后再构建镜像工程
            ```

* 问题汇总
    1. 在构建过程中，会提示某些包install失败
        + 解决方法:
            1. 登录到 apt_proxy, 即在浏览器上输入 192.168.4.40:5000, 登录到 apt cache上,在 File Station 中搜索 安装失败的包的名字，搜索出来后，把对应包的文件夹和安装文件整个删掉。 然后再跑 image-builder 的构建脚本
        + 原因:
            在 apt cache 上的包有可能在拉取的过程中失败，成为残缺文件，所以在构建image过程中，校验失败
