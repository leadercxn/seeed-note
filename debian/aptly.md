# apt-get

## 参考链接
[Creating and hosting your own deb packages and apt repo](https://earthly.dev/blog/creating-and-hosting-your-own-deb-packages-and-apt-repo/#keeping-your-private-key-secure)


## 自制 .deb 文件
1. 文件夹命名
    <package-name>_<version>-<release-number>_<architecture>
    demo:
        hello_0.0.1-1_amd64
        or
        hello_0.0.1-1_all
2. 文件夹目录
    ```demo
        hello_0.0.1-1_amd64
        ├── DEBIAN
        │   └── control     # .deb文件说明
        └── usr
            └── bin
                └── hello   # bin文件

    ```
    + control 文件demo
        ```
            Package: hello-world    # 包的名称，apt-get remove的时候，要用这个name
            Version: 0.0.1
            Maintainer: example <example@example.com>
            Depends: libc6
            Architecture: amd64
            Homepage: http://example.com
            Description: A program that prints hello
        ```

3. 把文件夹制成 .deb 文件 命令
    - 针对二进制可执行文件
    ```sh
        dpkg-deb --build dir    # 针对二进制可执行文件
    ```
    - 庞大和复杂的 Source PackagePermalink
    ```sh
        dh_make
    ```

4. 解压控制文件
    ```sh
        dpkg-deb -e drcom-pum_1.0-0ubuntu1~ppa1~jaunty1_i386.deb drcom/DEBIAN  # 解压程序文件
        dpkg-deb -x drcom-pum_1.0-0ubuntu1~ppa1~jaunty1_i386.deb drcom # 解压控制文件
    ```



## 创建 apt-repository -- 在服务器上搭建 apt-get install 下载列表
1. 创建apt-repo目录
    ```sh
        apt-repo/
        ├── dists
        │   └── stable
        │       ├── main
        │           └── binary-amd64
        └── pool
            └── main
                └── hello_0.0.1-1_amd64.deb
    ```
2. 把自生成的 .deb文件放到 apt-repo/pool/main/ 目录下
3. 生成 Packages 文件，该 Packages 文件包含该apt-repo包含的所有可用的 packages
    ```sh
        cd apt-repo
        dpkg-scanpackages --arch amd64 pool/ > dists/stable/main/binary-amd64/Packages
    ```
4. 打包 Packages
    ```
        cat dists/stable/main/binary-amd64/Packages | gzip -9 > dists/stable/main/binary-amd64/Packages.gz
    ```
5. 生成 Release 文件，该 Release 文件包含一些 验证SHA1，SHA256信息
    + 生成 Release 的脚本
        ```sh
            echo '#!/bin/sh
            set -e

            do_hash() {
                HASH_NAME=$1
                HASH_CMD=$2
                echo "${HASH_NAME}:"
                for f in $(find -type f); do
                    f=$(echo $f | cut -c3-) # remove ./ prefix
                    if [ "$f" = "Release" ]; then
                        continue
                    fi
                    echo " $(${HASH_CMD} ${f}  | cut -d" " -f1) $(wc -c $f)"
                done
            }

            cat << EOF
            Origin: Example Repository
            Label: Example
            Suite: stable
            Codename: stable
            Version: 1.0
            Architectures: amd64 arm64 arm7 armhf
            Components: main
            Description: An example software repository
            Date: $(date -Ru)
            EOF
            do_hash "MD5Sum" "md5sum"
            do_hash "SHA1" "sha1sum"
            do_hash "SHA256" "sha256sum"
            ' > apt-repo/../generate-release.sh && chmod +x generate-release.sh
        ```
    + 生成 Release 文件
        ```sh
            cd apt-repo/dists/stable

            ./../../../generate-release.sh > Release
        ```



6. 用 python3 创建一个http的服务器
    + cd apt-repo/../
    + python3 -m http.server


7. 在同一环境下，新建一个终端
    + 添加 apt-get 的 source.list
        ```sh
            echo "deb [arch=amd64] http://0.0.0.0:8000/apt-repo stable main" | sudo tee /etc/apt/sources.list.d/example.list
        ```
    + 更新 apt-get 源
        ```
            sudo apt-get update --allow-insecure-repositories
        ```
    + 安装软件包
        ```
            sudo apt-get install hello-world
        ```


## 签名 apt-repo
1. 创建一对新的 Public/Private PGP Key
    + 新建一个 batch 脚本
        ```sh
            echo "%echo Generating an example PGP key
            Key-Type: RSA
            Key-Length: 4096
            Name-Real: example
            Name-Email: example@example.com
            Expire-Date: 0
            %no-ask-passphrase
            %no-protection
            %commit" > apt-repo/../example-pgp-key.batch
        ```
    + 创建一个临时的文件夹
        ```sh
            export GNUPGHOME="$(mktemp -d apt-repo/../pgpkeys-XXXXXX)" #重定义 GNUPGHOME

            gpg --no-tty --batch --gen-key  apt-repo/../example-pgp-key.batch   #gpg生成密钥所需的文件放在刚创建的 pgpkeys-XXXXXX 目录下
        ```
    + 查看 GNUPGHOME 目录下的 private-key 目录
        ```
            ls "$GNUPGHOME/private-keys-v1.d"
        ```
    + 查看 gpg 的key
        ```
            gpg --list-keys
        ```
2. 创建 publich key
    + gpg生成 publich key
        ```sh
            gpg --armor --export example > apt-repo/../pgp-key.public   #利用 "example" 标识名生成 publich key
        ```
    + 验证是否只是导出了一个publich key
        ```
            cat apt-repo/../pgp-key.public | gpg --list-packets
        ```
3. 生成 private key
    + gpg生成 private key
        ```sh
            gpg --armor --export-secret-keys example > apt-repo/../pgp-key.private #利用 "example" 标识名生成 private key
        ```

4. 签名 Release
    + 倒入备份的 private key
        ```sh
            cat apt-get/../pgp-key.private | gpg --import
        ```
    + 签名
        ```sh
            cat apt-repo/dists/stable/Release | gpg --default-key example -abs > apt-repo/dists/stable/Release.gpg
        ```
    + 
        ```sh
            cat apt-repo/dists/stable/Release | gpg --default-key example -abs --clearsign > apt-repo/dists/stable/InRelease
        ```

5. 在同一环境下，新建一个终端
    + 带签名的source.list
        ```sh
            echo "deb [arch=amd64 signed-by=$HOME/github/aptly-test/pgp-key.public] http://0.0.0.0:8000/apt-repo stable main" | sudo tee /etc/apt/sources.list.d/example.list
        ```
    + 查看服务器的 private key
        ```
            curl http://0.0.0.0:8000/pgp-key.private
        ```

## 利用 github page作为网页服务器储存相关的 deb 或 工具包
* 搭建 github page
    + [GitHub Pages 搭建教程](https://sspai.com/post/54608)
    + 个人搭建大概总结
        1. 在个人github上新建 repository
        2. 新建完仓库之后，新建.md文件，demo: README.md ，假如没有md文件，github page生成的网页 xxx.git.io 会弹出404 error
        3. 选择 Settings , 在左边边栏选择 Pages
        4. 选择 github page 仓库选择的 git repository 分支，个人默认是 master 分支
        5. 路径选择 /(root)
        6. 选择一个主题 Theme Chooser
        7. 到此完成，无需填写 Custom domain ,这个 Custom domain 相当于重映射，通过自定义的域名登陆 github page
        8. 稍等2～3分钟，点击弹出的 https://leadercxn.github.io/aptly-test/ 链接，即可跳转到该 github page，可看到 README.md 文件内容
        9. git pull 和 git push都可以更新 git repository 和 github page的内容，git repository 的更新同步到 github page需要2～3分钟

* 把上面本地生成的 apt-repo 文件夹上传到github，并生成对应 github page,至此 github page即可充当远程服务器
* 更新 本地PC 的 apt-get 的源
    1. 不带签名的source.list
        ```sh
            deb [arch=armhf] https://leadercxn.github.io/aptly-test/apt-repo/ stable main
        ```
    2. 带签名的source.list, 其中 pgp-key.public 是上面过程生成的 public.key
        ```sh
            deb [arch=amd64 signed-by=/home/cxn/github/aptly-test/pgp-key.public] https://leadercxn.github.io/aptly-test/apt-repo/ stable main
        ```


## aptly 的使用
* 相关链接
    [APTLY 官方网站](https://www.aptly.info/doc/aptly/mirror/create/)

## aptly mirror 
* mirror 远程仓库
    1. 导入密钥
        ```sh
            gpg --no-default-keyring --keyring /usr/share/keyrings/ubuntu-archive-keyring.gpg --export | gpg --no-default-keyring --keyring trustedkeys.gpg --import
        ```
    2. mirror create
        ```sh
            aptly mirror create -architectures=amd64 -with-sources=false -with-udebs=false hello-world  https://leadercxn.github.io/aptly-test/apt-repo/ stable #在 leadercxn 的 github 上存放 aptly-test 仓库, -architectures=amd64 这个得根据自己的仓库repo的结构来添加，不添加限制，使用默认的话，有可能在 mirror update 阶段出现 "ERROR: unable to update: no candidates" 的错误
        ```
        + 因为在 aptly-test 仓库上保存有 pgp-key.public,所以需要导入远程仓库的 public-key,导入 publich-key 之后，即可aptly create
            ```sh
                wget -O - https://leadercxn.github.io/aptly-test/pgp-key.public | gpg --no-default-keyring --keyring trustedkeys.gpg --import
            ```
    3. mirror update
        ```sh
            aptly mirror update hello-world
        ```
    4. mirror drop
        ```sh
            aptly mirror drop hello-world
        ```

## aptly repo
* repo 本地仓库
    1. 创建本地仓库
        ```sh
            aptly repo create hello-world
            aptly repo create repo-local from snapshot snap-mirror
        ```
    2. 从 mirror 中导入 deb 包
        ```sh
            aptly repo import <src-mirror> <dst-repo> <package-query>
            
            demo:   aptly repo import hello-world hello-world Name  # Name 所有非空 packages
        ```
    3. 显示 repo 的详细信息
        ```sh
            aptly repo show -with-packages hello-world  # 把repo的包含的packages都展示了出来
        ```
    4. 添加包
        ```sh
            aptly repo add <name> <package file>|<directory> ...
        ```
    5. 删除包
        ```
            aptly repo remove <name> <package-query> .
        ```

## aptly snapshot
* snapshot , 个人理解: repo里面的 packages列表 和 一些 dependencies 依赖关系
    1. 创建 snapshot
        - 根据 mirror 来创建
            ```sh
                aptly snapshot create snap-mirror from mirror hello-world
            ```
        - 根据 repo 来创建
            ```sh
                aptly snapshot create snap-repo from repo hello-world
            ```
        - 创建空的 snapshot
            ```sh
                aptly snapshot create <name> empty
            ```
    2. 列出所有的 snapshot 
        ```sh
            aptly snapshot list
        ```
    3. 显示 snapshot 的详细信息
        ```sh
            aptly snapshot show snap-repo
        ```
    4. 对比两个 snapshot
        ```sh
            aptly snapshot diff <name-a> <name-b>
        ```
    5. 丢弃 snapshot
        ```sh
            aptly snapshot drop snap-repo
        ```


## aptly publish
* publish 发布
    1. 推送本地的 repo-local
        ```sh
            aptly publish repo  -distribution="stable" -architectures="amd64"  repo-local
        ```
        - 免签推送
            ```sh
                aptly publish repo  -distribution="stable" -architectures="amd64"  -skip-signing  repo-local
            ```
    2. 展示所有发布的 repo 或 snapshot
        ```sh
            aptly publish list
        ```



## 应用场景
* 思路流程
    1. 创建 mirror
    2. 创建 snapshot from mirror 已创建的 mirror
    3. 创建 repo from snapshot 已创建的 snapshot
    4. 把相关的 .deb 包添加到 repo, aptly repo add ***.deb
    5. 发布 repo 到本地, 类似
        ```sh
            aptly publish repo  -distribution="stable" -architectures="amd64"  -skip-signing  repo-local

            aptly publish repo  -distribution="stable" -architectures="amd64,armhf"  repo-local
        ```
    6. 把 ~/.aptly/publish 目录下的内容推送到 github，通过 git 来推送到 github page
    7. 后面开发，通过 aptly repo add deb 的命令往 repo 添加 deb 包，然后通过 aptly publish update <distribution> ( 该distribution类似于上面的stable) 更新到本地 ～/.aptly/publish ,最后通过 git push 更新到github page
    8. 在 系统上，添加 /etc/apt/sources.list.d/example.list 文件，添加源, 类似`deb [arch=amd64 signed-by=/home/cxn/github/aptly-test/pgp-key.public]  https://leadercxn.github.io/aptly-test/apt-repo/ stable main`  ，后 apt-get update 更新源，后通过 apt-get install 来安装 github page 上的安装包。

* 注意，一些坑
    1. aptly publish 的时候，可以选择免签和签名
        - 免签的话，加上 -skip-signing 选项字
            ```sh
                aptly publish repo  -distribution="stable" -architectures="amd64"  -skip-signing  repo-local    # -skip-signing 表示免签
            ```
            在 第三方使用 apt source.list 的时候，添加类似如下字段，不需要signed-by
                ```sh
                    deb [arch=amd64]  https://leadercxn.github.io/aptly-test/apt-repo/ stable main
                    sudo apt-get update --allow-insecure-repositories   # --allow-insecure-repositories 表示允许免签的安装包 ,不然提示 “Release is not signed” 出错
                ```
        - 签名的话
            由于个人在使用的时候，新建了 gpg-key.private 和 gpg-key.publish 来签名发布的 Release,用来生成 gpg key 的一堆文件放在临时新建的目录下，此时终端环境相关的环境变量得指向对应的目录,并导入相关的 private key. (PS: 但这样的操作在192.168.1.78 环境下还是提示出错)
                ```sh
                    export GNUPGHOME=/home/cxn/github/aptly-test/pgpkeys-q8VtZJ

                    cat ~/github/aptly-test/pgp-key.private | gpg --import
                ```
            后再来发布 aptly publish repo。
            在 第三方使用 apt source.list 的时候，添加类似如下字段，需要signed-by 和 privat key 对应的 publish key
                ```sh
                    deb [arch=amd64 signed-by=/home/cxn/github/aptly-test/pgp-key.public]  https://leadercxn.github.io/aptly-test/apt-repo/ stable main
                ```
            又或者把仓库的 publish-key 下载下来，通过 apt-key add 命令添加,这样的话， source.list 就不需要 signed-by,更建议使用这种方法
                ```sh
                    deb [arch=amd64]  https://leadercxn.github.io/aptly-test/apt-repo/ stable main

                    curl https://leadercxn.github.io/aptly-test/pgp-key.public | sudo apt-key add -
                ```



