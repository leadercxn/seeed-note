# Zephyr-BLE


## 数据结构

struct bt_data {
	u8_t type;
	u8_t data_len;
	const u8_t *data;
};

BT_LE_ADV_CONN_NAME 
    ==> BT_LE_ADV_PARAM( BT_LE_ADV_OPT_CONNECTABLE | BT_LE_ADV_OPT_USE_NAME, \
					     BT_GAP_ADV_FAST_INT_MIN_2, \
					     BT_GAP_ADV_FAST_INT_MAX_2, NULL)
                         
        ==> BT_LE_ADV_PARAM(_options, _int_min, _int_max, _peer) \
                            ((struct bt_le_adv_param[]) {   \
                            { \
                                .id = BT_ID_DEFAULT, \
                                .sid = 0, \
                                .secondary_max_skip = 0, \
                                .options = (_options), \
                                .interval_min = (_int_min), \
                                .interval_max = (_int_max), \
                                .peer = (_peer), \
                            } })

## API函数

1. 广播
    bt_le_adv_start(BT_LE_ADV_CONN_NAME, ad, ARRAY_SIZE(ad), NULL, 0);
        ==> bt_le_adv_start_legacy(param, ad, ad_len, sd, sd_len);
            ==> 1. adv = adv_new_legacy()
                    ==> return &bt_dev.adv;
                2. le_adv_set_random_addr (adv, param->options, dir_adv, &set_param.own_addr_type);
                3. bt_addr_le_copy(&adv->target_addr, BT_ADDR_LE_ANY);

## 逻辑
1. 初始化
bug1:
    bt_enable < == bt_init <== hci_init <== common_init <= bt_hci_cmd_send_sync(BT_HCI_OP_RESET, NULL, &rsp);

bug2:
    





## 文件
1. hci.h
    ```C
        /* HCI BR/EDR link types */
        #define BT_HCI_SCO                              0x00
        #define BT_HCI_ACL                              0x01
        #define BT_HCI_ESCO                             0x02

        /* OpCode Group Fields */
        #define BT_OGF_LINK_CTRL                        0x01
        #define BT_OGF_BASEBAND                         0x03
        #define BT_OGF_INFO                             0x04
        #define BT_OGF_STATUS                           0x05
        #define BT_OGF_LE                               0x08
        #define BT_OGF_VS                               0x3f

        /* Construct OpCode from OGF and OCF */
        #define BT_OP(ogf, ocf)                         ((ocf) | ((ogf) << 10))
    ```
0x0c03
    demo:
        BT_HCI_OP_LE_READ_BUFFER_SIZE           BT_OP(BT_OGF_LE, 0x0002) => 0x0002|(0x08 << 10) ==> 0x2002
        BT_HCI_OP_LE_SET_RANDOM_ADDRESS         BT_OP(BT_OGF_LE, 0x0005) => 0x0005|(0x08 << 10) ==> 0x2005
