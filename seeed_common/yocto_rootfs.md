# Yocto-rootfs

## 修改文件系统的属性
* 方法
    1. 在image的recipe中使用IMAGE_FEATURE变量中添加 "read-only-roofts"
        ```sh
            IMAGE_FEATURES += "read-only-rootfs"
        ```
    2. 在build/conf/local.conf文件中添加
        ```sh
            EXTRA_IMAGE_FEATURES = "read-only-rootfs"
        ```
