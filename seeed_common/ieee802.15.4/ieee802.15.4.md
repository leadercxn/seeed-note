# IEEE802.15.4

* 物理帧格式
    ｜ 前导码 ｜ 起始分隔符 ｜ 物理头端 ｜ payload ｜

    前导码:    4B 用于符号同步
    起始分隔符: 1B 用于帧同步
    物理头端:   1B  length
    payload:   <= 127B  负载数据

* MAC帧
    * 帧的类型: 信标帧、数据帧、确认帧、mac命令帧
    1. 常规帧格式
    ｜ 帧控制域 ｜ 帧序列号 ｜ 目标id ｜ 目标地址 ｜ 源id ｜ 源地址 ｜ 附加安全头部｜ 帧负载 ｜ FCS校验 ｜
                         |                地址域             |
    * 不同类型的帧格式
        1. 信标帧 -- Beacon Frame
        ｜ 帧控制域 ｜ 帧序列号 ｜ 地址域 ｜ 附加安全头部｜ 超帧描述 ｜ GTS 分配释放信息 ｜ 待发数据目标地址信息 ｜帧负载 ｜ FCS校验 ｜

        2. 数据帧 -- Data Frame
        ｜ 帧控制域 ｜ 帧序列号 ｜ 地址域 ｜ 附加安全头部 ｜帧负载 ｜ FCS校验 ｜

        3. 确认帧 -- Acknowledgment Frame
        ｜ 帧控制域 ｜ 帧序列号 ｜ FCS校验 ｜

        4. 命令帧 -- MAC Command Frame
        ｜ 帧控制域 ｜ 帧序列号 ｜ 地址域 ｜ 附加安全头部 ｜ 命令帧ID ｜ 命令帧负载 ｜FCS校验 ｜

* 

## 名词的理解
* 
	+ promiscuous mode:	混杂模式又叫偷听模式，指一台机器的网卡能够接收所有经过它的数据流，而不论其目的地址是否是它。

## 


## kernel 相关宏
* Hardware flags
    + IEEE802154_HW_TX_OMIT_CKSUM:  Indicates that xmitter will add FCS on it's own.
    + IEEE802154_HW_LBT:            Indicates that transceiver will support listen before transmit.
    + IEEE802154_HW_CSMA_PARAMS:    Indicates that transceiver will support csma parameters (max_be, min_be, backoff exponents).
    + IEEE802154_HW_FRAME_RETRIES:  Indicates that transceiver will support ARET frame retries setting.
    + IEEE802154_HW_AFILT:          Indicates that transceiver will support hardware address filter setting.
    + IEEE802154_HW_PROMISCUOUS:    Indicates that transceiver will support promiscuous mode setting.
    + IEEE802154_HW_RX_OMIT_CKSUM:  Indicates that receiver omits FCS.
    + IEEE802154_HW_RX_DROP_BAD_CKSUM:  Indicates that receiver will not filter frames with bad checksum.

## 数据结构
*  mac802154.h
    ```c
        struct ieee802154_hw {
            /* filled by the driver */
            int	extra_tx_headroom;
            u32	flags;
            struct	device *parent;
            void	*priv;

            /* filled by mac802154 core */
            struct	wpan_phy *phy;
        };
    ```

* cfg802154.h
    ```c
        struct wpan_phy_supported {
            u32     channels[IEEE802154_MAX_PAGE + 1],cca_modes, cca_opts, iftypes;
            enum    nl802154_supported_bool_states lbt;
            u8      min_minbe, max_minbe, min_maxbe, max_maxbe, min_csma_backoffs, max_csma_backoffs;
            s8      min_frame_retries, max_frame_retries;
            size_t  tx_powers_size, cca_ed_levels_size;
            const s32   *tx_powers, *cca_ed_levels;
        };

        struct wpan_phy {
            const void *privid;

            u32 flags;
            u8 current_channel;
            u8 current_page;
            struct wpan_phy_supported supported;
            s32 transmit_power;
            struct wpan_phy_cca cca;

            __le64 perm_extended_addr;

            /* current cca ed threshold in mBm */
            s32 cca_ed_level;

            /* PHY depended MAC PIB values */

            /* 802.15.4 acronym: Tdsym in usec */
            u8 symbol_duration;
            /* lifs and sifs periods timing */
            u16 lifs_period;
            u16 sifs_period;

            struct device dev;

            /* the network namespace this phy lives in currently */
            possible_net_t _net;

            char priv[] __aligned(NETDEV_ALIGN);
        };
    ```
