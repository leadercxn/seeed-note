# STM32MP135x

## task-list
0. 启动流程
    - 熟悉，对比跟 157 的启动差异

1. tf-a 适配移植
    - 设备树
    - 电源管理

2. optee 适配移植
    - 设备树

3. u-boot
    - 设备树
    - npi-common.h 移植

4. kernel
    - 驱动
        + 电源管理IC, STPMIC1DPQR
        + 以太网 PHY/ RMII
        + USB type-A/type-C
        + Camera
        + LCD RGB 显示屏
        + button, led
        + SD 卡

5. 40 PIN
    1. I2C  -- 需要器件配合
    2. SPI  -- 需要器件配合
    3. UART
    4. QSPI  -- 需要器件配合
    5. TIMER-CH
    6. SAI  -- 需要器件配合
    7. GPIO



# Schematic
1. PMIC-STPMIC1DPQR
    I2C-address: 0x33
2. WM8960
    I2C-address: 0x34

