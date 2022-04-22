# Zephyr

## 实操
* 安装python依赖
    1. pip3 install --user -U west
    2. echo 'export PATH=~/.local/bin:"$PATH"' >> ~/.bashrc
    3. source ~/.bashrc
* 获取 zephyr 源码
    1. west init ~/zephyrproject
    2. cd ~/zephyrproject
    3. west update
* 导出 zephyr  cmake 包
    1. west zephyr-export

* 安装其他工具
    1. 安装ninja
    2. pip3升级, python3 -m pip install --upgrade pip

* 声明了额外的Python依赖项(required)
    1. pip3 install --user -r ~/zephyrproject/zephyr/scripts/requirements.txt

* 安装工具
    1. cd ~ && wget https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.13.0/zephyr-sdk-0.13.0-linux-x86_64-setup.run
    2. chmod +x zephyr-sdk-0.13.0-linux-x86_64-setup.run
    3. 安装sdk ,  ./zephyr-sdk-0.13.0-linux-x86_64-setup.run -- -d ~/zephyr-sdk-0.13.0
    4. 安装udev
        ```sh
            sudo cp ~/zephyr-sdk-0.13.0/sysroots/x86_64-pokysdk-linux/usr/share/openocd/contrib/60-openocd.rules /etc/udev/rules.d
            sudo udevadm control --reload
        ```

## 调试
* west 相关的使用
    1. west build -t menuconfig

* cmake && ninja 相关的使用
    1. 

* prj.conf


## 设备树
* 添加 .dts 到自定义应用程序
    1. 在板级目录（ -DBOARD_ROOT ）下创建bsp架构的文件夹（eg: x86,arm,etc）,在架构目录下创建自定义板子名称的目录（-b my_custom_board）,在 自定义板子名称的目录 下创建 .dts文件

* 添加 .overlay 的方式
    1. 在编译命令时，使用 `-DDTC_OVERLAY_FILE="file1.overlay;file2.overlay"` 指定 .overlay 文件
    2. CMakeLists.txt 中定义
        set() 设置
    3. 把 `DTC_OVERLAY_FILE` 宏放到系统环境变量
    4. 创建 `boards/<BOARD>_<revision>.overlay` ,在应用程序目录下创建该文件。若存在 `boards/<BOARD>.overlay` ,以上两个文件会merge
    5. 创建 `boards/<BOARD>.overlay` 文件
    6. 在应用程序目录 创建 `<BOARD>.overlay`
    7. 在应用程序目录 创建 `app.overlay`

* 最终编译生成的 zephyr.dts
    1. 经历 .dts , .dtsi 和 .overlay 之后,最终会综合以上三种类型的设备树文件，会生成 build/zephyr/zephy.dts ,可在此检查自己添加的设备树节点是否添加成功

* compatiable


