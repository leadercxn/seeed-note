# node 技术文档 v1.2

## E5
1. 升级固件组成结构: app_dfu_version_date.dfu
    app_dfu_version_date.dfu(sfb) = mete_data + app_dfu(暗文)

    + mete_data = dfu_header_t + nonce + app_sha256(32 B)摘要
    + dfu.settings & app_settings, 即 dfu_header_t & app_header_t
        ```C
            typedef struct
            {
                uint32_t magic_word;
                uint32_t sw_version;
                uint32_t length;
                uint32_t crc32;
            } dfu_header_t;
        ```
2. 分区

    | 地址          |     信息               |
    | ------------ | ---------------------- |
    |   0x00000000 |   bootloader(SBSFU)    |
    |   0x00000000 |     app                |



## UART-BLE
1. 根据 EUI 来设置名字
2. 数据包分类:
    + 请求包
        - AT + XXXX = ?
    + 配置包
        - AT + XXXX?
    + 升级
        - AT + OTA?
3. 升级
    + 采用 Y-modem


## lora
1. 常规lorawan接口
    + 确认包:      LoRaConfirmedPacketSend(port, confirm_nb, buff, buff_len);
        - confirm_nb = 1，再回调里再发送
    + 非确认包:    LoRaUnconfirmedPacketSend(port, buff, buff_len);
    + 保存帧号:    LoRaUplinkCounterSet(fcnt);
2. lorawan包头
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
3. 端口号
    + 上电包 -- 端口号 6
    + 信号测试包 -- 端口号 4
    + 心跳数据包 -- 端口号 2
    + classA/B/C -- 端口号 7/8/9
4. 接口使用
    + 数据包统一采用 2C + 1N
5. 事件包
    + 上电包
        1. 参考一代 <LoRaWan 设备数据说明>: hw-version, sw-version, upstream-interval, EUI,出厂时间,channel_mask
        2. sensor_type
    + 常规包 -- 沿用一代
        1. frame 格式

            |    channel   |    data id   |  data value  |
            | ------------ | ------------ |------------- |
            |      1B      |      2B      |      4B      |

        2. 组包

            |    frame1    |    frame2    |  frame3 ...  |  crc16 |
            | ------------ | ------------ |------------- |--------|
            |      7B      |      7B      |      7B      |    2B  |

        3. LSB 排序
    + 信号测试包
        1. 空包
        2. 不采用 2C + 1N
    + 下行包

            |    frame     |    crc16      |
            | ------------ | ------------- |
            |      11B     |      2B       |

6. 相关处理
    1. 沿用2C + 1N
    2. OTAA -- OTAA需要的信息配置烧录进去, EUI/APPEUI/APPSKEY
    3. ABP  -- 保留可配置接口


## spi flash
1. 文件

    |     file                  |          fun           |
    | ------------------------- | ---------------------- |
    |           factory.hex     |        出厂app          |
    | app_dfu_version.dfu       |        待升级app        |
    |       event_log.log       |       设备事件日志保存    |
    |       lora_fcnt.txt       |        上行帧号保存      |
    |       event_dfu.txt       |     拉取升级固件事件标记   |
    |       app_settings.txt    |     保存当前app的固件信息 |
    |       factory_param.txt   |     factory_parameter  |
    |       app_param.txt       |     app_parameter      |

    + event_dfu.txt 的数据结构
        ```C
            typedef enum
            {
                DFU_EVENT_NONE, //没有新的固件
                DFU_EVENT_NEW,  //有新的固件，且已完成解密&校验
            };
        ```

## sensor
1. device_type, 出厂EUI包含，二代固件不做任何处理
2. sensor type -- 对照一代的 <传感器类型及测量值ID汇总>
    + 需要在上电包中体现
3. mesurement ID -- 参考一代的 <LoRaWan 设备数据说明> 文档

## 总体
1. 出厂参数的数据结构 -- 即那些参数要通过配置烧录到片上flash
    ```C
        #define IDENTITY_DATA_LEN       84

        typedef struct
        {
            uint32_t magic_byte;
            uint32_t hw_version;
            uint32_t sw_version;

            uint8_t devEui[8];
            uint8_t appEui[8];
            uint8_t appKey[16];
            uint8_t nwkSKey[16];
            uint8_t appSKey[16];
            uint32_t devAddr;

            uint32_t report_interval;
        } __attribute__((__packed__)) fac_param_cfg_t;


        typedef union
        {
            identity_data_t data;
            uint8_t bytes[IDENTITY_DATA_LEN];      
        }identity_data_u;
    ```
2. 错误码 err_code -- 参考一代的 <LoRaWan 设备数据说明> 文档
3. 按键
    + 短按 -- 软复位
    + 长按5s -- 打开蓝牙，触发采集上行

4. 配置
    + uart-ble 配置
        * 上行间隔
        * 设备 EUI
        * APP EUI
        * 入网方式
            1. OTAA
            2. ABP
        * 频段设置
            1. EU868
            2. IN865
            3. US915
                - sub_band
            4. AU915
                - sub_band
            5. AS923
        * 传感器校准
            + cmd + 校准的参数

    + lora下行 配置
        * 重启
        * 修改上行间隔
5. adr开关接口预留


## 测试固件
1. 把spi-flash初始化little-fs 


## 参考链接
1. [lorawan 设备数据说明](https://thoughts.teambition.com/workspaces/618902eda8a0d900417e592c/docs/61890ce99d668f0001430e96)
2. [传感器类型及测量值ID汇总](https://shimo.im/sheets/XKq4MmjZnYFZJ1kN/MODOC)




## 问题
1. app_settings损坏的处理 -- done
    1. 损坏了无所谓，直接升级
2. factory.hex怎么放到spi-flash
    - planA
        通过 Y-modem 传进去，在 meta_data 添加flag 做区分
    - planB
        通过SPI烧录器怼进去
3. e5生产测试固件把spi-flash初始化little-fs -- done
4. OTAA入网失败，采样，发送就不用去做 -- done
5. ABP可以直接发送 -- done
6. 每次发单元，之前CAD -- done
7. ble 服务1对1 -- done
    - AT指令设置
8. lora发送前喂狗 -- done
9. adr开关接口预留 -- done

