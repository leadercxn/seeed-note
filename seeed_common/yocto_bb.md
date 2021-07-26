# Yocto-bb

## 函数
* bitbake支持使用函数来做相应的功能和任务，支持 [bb文件中函数实操](https://fulinux.blog.csdn.net/article/details/113871052)
    1. shell 函数。
        可以在函数后面加上 _append 或者 _prepend 关键词来规定执行的顺序
    2. Bitbake风格的python函数
    3. Python函数
    4. 匿名Python函数

* addtask 只能在recipe或者classes中使用
    ```sh
        poky]$ vim meta-mylayer/recipes-example/example/example_0.1.bb
            ...
            do_foo() {
                bbplain first
                fn  
            }

            fn_prepend() {
                bbplain second
            }

            fn() {
                bbplain third
                echo "third 3th"
            }

            do_foo_append() {
                bbplain fourth
            }

            addtask foo     # 把函数升级为任务
    ```
* deltask 将某个任务从task列表中删除
    应用场景：一个项目不想开源，只提供库文件和可执行程序，就不需要 do_compile
    ```sh
        do_compile() {
        }

        do_compile[noexec] = "1"
        or
        deltask do_compile
    ```


* 使用after或before，来规定任务的执行顺序
    ```sh
        see: meta-mylayer/recipes-myfunctions/myfunctions/myfunctions_0.1.bb
        python do_printdate() {
            DATE = "${@time.strftime('%Y%m%d', time.gmtime())}";
            bb.plain(d.getVar("DATE"));
        }

        addtask printdate after do_fetch before do_build

    ```
* 让任务每次都执行，不要第一次执行之后，要等到有修改才执行第二次。方框内 nostamp 的使用
    ```sh
        python do_printdate() {
            DATE = "${@time.strftime('%Y%m%d', time.gmtime())}";
            bb.plain(d.getVar("DATE"));
        }

        do_printdate[nostamp] = "1"   #加了这一行，注释不要加哦

        addtask printdate after do_fetch before do_build

        # nostamp的作用是，当设置为1时，告诉bitbake不要问任务生成时间戳文件。意味着任务始终执行
    ```
    类似方框内的关键词还有：
        noexec 不执行某函数
        number_threads 设置某个任务执行时县城数量的限制


## 获取源码

* 通过 git 方式获取
    ```
        recipes-learnyocto]$ cat learnyocto/learnyocto_git.bb 
        SRC_URI = "git://gitee.com/fulinux/learnyocto.git;protocol=https;branch=master \
                file://0001-learn-the-CFLAGS-variable-in-recipe.patch \
                "
    ```

* wget 方式获取
    ```sh
        SRC_URI = "http://example.com/foobar.tar.bz2;md5sum=4a8e0f237e961fd7785d19d07fdb994d \
		   http://example.com/abc.tar.bz2;md5sum=131h42hjg37e961fd7785d19d07f432f"
    ```
    md5sum可要可不要
    ```sh
        SRC_URI = "http://example.com/foobar.tar.bz2;name=foo"
        SRC_URI += "http://example.com/abc.tar.bz2;name=abc"
        SRC_URI[foo.md5sum] = 4a8e0f237e961fd7785d19d07fdb994d
        SRC_URI[abc.md5sum] = 131h42hjg37e961fd7785d19d07f432f
    ```
* 存在本地的源码文件
    ```sh
        mkbootimg]$ ls
        mkbootimg  mkbootimg-native_git.bb
        mkbootimg]$ cat mkbootimg-native_git.bb
        #省略部分内容
        FILESEXTRAPATHS_prepend := "${THISDIR}/:"
        SRC_DIR = "mkbootimg"

        SRC_URI = "file://mkbootimg/"
        S = "${WORKDIR}/mkbootimg"
        ... #省略部分内容
    ```


## 解压压缩包
    bitbake支持的压缩包文件格式有：*.Z  .z  .gz  .xz  .zip  .jar  .ipk  .rpm  .srpm  .deb  .bz2
    ```
        SRC_URI = "http://example.com/foobar.tar.bz2;unpack=1"
    ```

poky]$ git clone https://gitee.com/fulinux/meta-mybsp.git
poky]$ source oe-init-build-env 
build]$ bitbake-layers add-layer ../meta-mybsp #加入conf/bblayers.conf中
