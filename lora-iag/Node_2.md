# Node-2

## PlanA -- `弃用`
### nrf52840
* 芯片特性
    1. 1024 kB flash 和 256KB RAM
    2. 协议栈 SDK16-s140_7.0.1  156K--0x27000,
    3. bootloader  24/32 KB

-------------------------------------------
### nrf52840 flash的分区
* 分区信息

    | 地址          |     信息     |
    | ------------ | ------------ |
    |   0x00000000 |  Softdevice  |
    |   0x00027000 |    app       |


--------------------------------------------
--------------------------------------------



## PlanB
-------------------------------------------
## SPI flash
* 分区信息 -- `弃用`
    | 地址          |     信息               |
    | ------------ | ---------------------- |
    |   0x00000000 |     dfu                |
    |   0x00000000 |     dfu.header         |
    |   0x00000000 |     log                |
    |   0x00000000 |     factory_parameter  |
    |   0x00000000 |     app_parameter      |
    |   0x00000000 |     fcnt               |
    |   0x00000000 |     sensors_cache      |

* app_dfu_version_date.dfu 升级固件结构
    app_dfu_version_date.dfu(sfb) = mete_data + app_dfu(暗文)
    + mete_data = dfu_header_t + nonce + app_sha256(32 B)摘要


* 数据结构
    1. dfu.settings & app_settings
        ```C
            typedef struct
            {
                uint32_t magic_word;
                uint32_t sw_version;
                uint32_t length;
                uint32_t crc32;
            } dfu_header_t;
        ```

* 文件

    |     file                  |          fun           |
    | ------------------------- | ---------------------- |
    |           factory.hex     |        出厂app          |
    | app_dfu_version_date.dfu  |        待升级app        |
    |       packets_cache.data  |       数据包缓存         |
    |       event_log.log       |       设备事件日志保存    |
    |       fcnt.txt            |        上行帧号保存      |
    |       event_dfu.txt       |     拉取升级固件事件标记   |
    |       app_settings.txt    |     保存当前app的固件信息 |
    |       app_param_dfu.bin   |  待升级的配置app参数(待定) |

    + event_dfu.txt 的数据结构
        ```C
            typedef enum
            {
                DFU_EVENT_NONE, //没有新的固件
                DFU_EVENT_NEW,  //有新的固件，且已完成解密&校验
            };
        ```

-------------------------------------------
## E5
* nor-flash 分区信息
    | 地址          |     信息               |
    | ------------ | ---------------------- |
    |   0x00000000 |     bootloader         |
    |   0x00000000 |     app                |
    |   0x00000000 |     factory_parameter  |
    |   0x00000000 |     app_parameter      |



-------------------------------------------
## 实现的细节
* node_2 的组包方式
    + lorawan 的接口
        1. 确认包:      LoRaConfirmedPacketSend(port, confirm_nb, buff, buff_len);  //confirm_nb = 1，再回调里再发送
        2. 非确认包:    LoRaUnconfirmedPacketSend(port, buff, buff_len);
        3. 保存帧号:    LoRaUplinkCounterSet(fcnt);
    + lorawan 包

        MHDR(1) + FHDR(7) + Port(1) + PAYLOAD(N B) + MIC(4)

        其中

            FHDR = DevAddr(4B) + FCtrl(1B) + FCnt(2B) + FOpts(可选)
        展开形成的数据结构
        ```C
            typedef struct
            {
                //MHDR
                uint8_t     mhdr;
                //FHDR
                uint32_t    devaddr;

                uint8_t     fops_len    : 4;
                uint8_t     fpending    : 1;
                uint8_t     ack         : 1;
                uint8_t     adr_ackreq  : 1;
                uint8_t     adr         : 1;

                uint16_t    fcnt;
                //Port
                uint8_t     fport;
            } __attribute__((__packed__)) pkt_header_t;
        ```
        + PAYLOAD(N B)
            1. 字节序排序法
                ```C
                    protocol_version (1 B) + interval （2B）+ datarate(4b) + tx_power(4b) + app_version(2B) + battery(2B) + sensors_data(N B) + threshold_value(N B) + error (1B)
                ```
            2. protobuf 打包
                ```

                ```
            3. 其他
        + 端口号
            1. 上电包 -- 端口号 --  device_type(lorawan,lora pp,sensorhub),sensor_type ,必须保证到达 ; 如果每一包包含有device_type,则不需要
            2. Class类型包 -- 端口号,不同类型的Class，不同的端口号
            3. 入网申请包 -- 端口号
            4. 搜频包 -- 端口号 -- 待确认
            5. 信号测试包 -- 端口号
            6. 命令包 -- 端口号
            7. 心跳包 -- 端口号
        + 事件包
            1. 心跳数据包 -- 确认
            2. 自检包 -- 确认 , PS: 每隔24h发一包
            3. 搜频包 -- 确认
            4. 信号测试包 -- 确认
            5. 异常包（采集出错,拆卸） -- 确认
            
            

* UART-BLE
    * 广播数据采样
        1.  广播包 adv_data(Max 31B)  -- `保留接口`
            ```C
                uint8_t adv_data[] =
                {
                    0x02, 0x01, 0x06,
                    0x1a, 0xff,
                    0x4C, 0x00, 0x02, 0x15,                         //ibeacon标识
                    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, // UUID
                    0x00, 0x01,                                     // Major Value
                    0x00, 0x01,                                     // Minor Value
                    0xC5
                };
            ```
        2. 响应包 rsp_data(Max 31B)  -- `保留接口`
            ```C
                uint8_t rsp_data[] =
                {
                    0x03, 0x03, 0xCE, 0x6C,                         //供app识别
                    26, 0x16, 0xCE, 0x6C,
                    0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, //EUI后8位
                    0x00,0x00                                       //硬件版本号
                    0x00,0x00,                                      //固件版本号
                    0x00,0x00,                                      //设备电量
                    0x00,0x00,                                      //BLE功率
                    0x00,0x00,                                      //BLE间隔
                    0x00,                                           //lora 功率
                    0x00,                                           //lora sf & adr, 高四位sf,低1位adr
                    0x00, 0x00,                                     //lora 间隔
                    0x00,                                           //保留
                };
            ```
        3. mac 地址 需要设置: SN 后 6字节   -- `保留接口`
        4. 设置 目标服务的 128bit UUID: 0x7364
        5. 数据包
            * 类型
                + 链接包
                + 配置包
                + 升级包
                + 命令包
                + other
            * 格式
                + 总包格式
                    包头 1B + 包头反码 1B + 随机数 1B + 暗文长度 2B + 暗文 nB + crc32 2B
                + 明文
                    packet_type 1B + 长度 2B + 数据 nB
        6. 加密的方式
            1. 以 16B AppKey ^ 随机数 => 密钥, 采用 AES128 的加密方式,进行蓝牙数据交互
        7. Y-modem 没有加密功能
            1. [Y-modem](https://blog.csdn.net/u012953523/article/details/78911800)
            2. 

        


* 配置阶段
    1. 串口流配置
        + 参考一代的？
            个人理解:   扫码枪获取二维码 => PC工具 => 访问平台 => 拉取配置数据 => uart发送到 MCU ?
        + 方法方面:
            PC端工具生成.bin文件，JLink 怼到对应的 MCU-flash 片区？        

* 功耗处理
    1. MCU主频降低
    2. 进入睡眠前，串口，i2c等外设IO恢复到默认状态

## app使用到的数据结构

1. 错误码   -- 参考一代的
    ```C

    ```

2. 出厂配置数据 -- lora相关的参考一代
    ```C
        typedef struct
        {
            uint8_t lora_eui[8];
            uint8_t lora_app_eui[8];
            uint8_t lora_net_id[3];
            uint8_t lora_appkey[8];
            uint8_t lora_nwkskey[8];
            uint8_t lora_appskey[8];
            uint8_t lora_app_nonce[3];
            uint8_t lora_dev_nonce[2];
            uint8_t lora_dev_addr[4];       //来自MCU本身

            uint8_t lora_channel_mask[8];
            uint8_t lora_band;
            uint8_t lora_data_rate;
            bool    lora_adr;
            int32_t lora_tx_power;

            uint16_t    app_ble_interval;
            uint8_t     app_ble_tx_power;
            uint8_t     app_report_rty;         //确认包失败重发的次数
            uint32_t    app_report_interval;    //lora上行间隔,单位s
            uint16_t    app_hw_version;         //硬件版本号,MSB-8bit区分大版本,LSB-8bit区分小版本
            uint32_t    reserve;
        } factory_cfg_param_t;
    ```
    




## 会议问题
1. 升级保留标记在spi-flash上，用来确认是否升级完成 -- done
2. 配置参数放在片上的nor-flash -- done
3. 蓝牙名字使用EUI命名 -- done
4. 打包协议格式参考一代
5. 采样多次失败，设置错误码
6. 蓝牙配置模式的使用，ble-task 内容  -- done
7. 可配置的参数具体
8. 电源管理，异常管理，传感器采样次数，按键，led指示
9. sensor-hub
10. 生产配置的数据结构


* 需要IAG提供
1. 节点错误码的代号枚举
2. 节点lora组包的具体数据结构
    1. sensor type
    2. mesurement ID
    3. device_type
3. sensor-hub ,即不同传感器的设备ID代号
4. 生产配置时，所用的数据结构 或 模版 或要设置什么参数，借来参考
    1. 

* 需和IAG确认
1. packets-cache 缓存的必要性
    1. 先发旧的，再发新的，因为平台不接受帧号倒退
    2. 不做
2. 节点时间戳的必要性
    1. 1代没用，之前有砍掉
3. 端口号的使用
    1. 使用
4. 事件包的类型，和逻辑
    1. 都是确认包
    2. 沿用 2C + 1N
5. 扫频功能的必要性
    1. 先不做
6. 软重启每次都要空中注册？
    1. OTAA -- OTAA需要的信息烧录进去, EUI/APPEUI/APPSKEY
    2. ABP  -- 保留接口
7. 按键的使用
    1. 长按5s蓝牙
    2. 按短软复位
8. 可配置参数的具体定案
    1. lora下行
    2. ble下行
9. 把出厂配置数据生成bin文件的格式,jlink烧录到指定的 nor-flash 地址
    1. 
10. lora payload 可以不用crc32（省 2B）
11. 配置参数上下阈值要合理
12. 程序跑死的措施
    1. 多久软重启一次，
13. 硬件看门狗的必要性？
    1. 讨论共识：有必要
14. 按键改成硬复位？外部硬件延时按键，按键还是接到rst引脚
    1. 不可靠
15. 看门狗
    1. lora发送前喂狗
16. 电池电量是否包含到每一帧的数据包中去？而不是只是在上电包中显示