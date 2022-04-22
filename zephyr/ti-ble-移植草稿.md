# 移植草稿


* old 的分析
    * lll 层的分析
        * platform平台
            * 特殊放在各个平台下的 c 文件
                1. lll_conn.c   //头文件在 lll-common 层
                2. lll_test.c   //在 lll-common 层有 ll-test.c
                3. lll.c        //头文件在 lll-common 层
            * 头文件的大概分析
                1. lll_adv_internal.h
                    ```C
                        struct pdu_adv *lll_adv_pdu_latest_get(struct lll_adv_pdu *pdu, u8_t *is_modified);
                        struct pdu_adv *lll_adv_data_latest_get(struct lll_adv *lll, u8_t *is_modified);
                        struct pdu_adv *lll_adv_scan_rsp_latest_get(struct lll_adv *lll,u8_t *is_modified);
                        struct pdu_adv *lll_adv_data_curr_get(struct lll_adv *lll);
                        struct pdu_adv *lll_adv_scan_rsp_curr_get(struct lll_adv *lll);
                    ```
                2. lll_adv.h
                    ```C
                        int lll_adv_init(void);
                        int lll_adv_reset(void);
                        void lll_adv_prepare(void *param);
                        struct pdu_adv *lll_adv_pdu_alloc(struct lll_adv_pdu *pdu, u8_t *idx);
                        void lll_adv_pdu_enqueue(struct lll_adv_pdu *pdu, u8_t idx);
                        struct pdu_adv *lll_adv_data_alloc(struct lll_adv *lll, u8_t *idx);
                        void lll_adv_data_enqueue(struct lll_adv *lll, u8_t idx);
                        struct pdu_adv *lll_adv_data_peek(struct lll_adv *lll);
                        struct pdu_adv *lll_adv_scan_rsp_alloc(struct lll_adv *lll, u8_t *idx);
                        void lll_adv_scan_rsp_enqueue(struct lll_adv *lll, u8_t idx);
                        struct pdu_adv *lll_adv_scan_rsp_peek(struct lll_adv *lll);

                        extern u16_t ull_adv_lll_handle_get(struct lll_adv *lll);
                    ```
                3. lll_clock.h
                    ```C
                        int lll_clock_wait(void);
                    ```
                4. lll_internal.h
                    ```C
                        int lll_prepare_done(void *param);
                        int lll_done(void *param);
                        bool lll_is_done(void *param);
                        int lll_clk_on(void);
                        int lll_clk_on_wait(void);
                        int lll_clk_off(void);
                        u32_t lll_evt_offset_get(struct evt_hdr *evt);
                        u32_t lll_preempt_calc(struct evt_hdr *evt, u8_t ticker_id, u32_t ticks_at_event);
                        void lll_chan_set(u32_t chan);
                        u8_t lll_entropy_get(u8_t len, void *rand);
                    ```
                5. lll_master.h
                    ```C
                        int lll_master_init(void);
                        int lll_master_reset(void);
                        void lll_master_prepare(void *param);
                    ```
                6. lll_prof_internal.h
                    ```C
                        void lll_prof_latency_capture(void);
                        void lll_prof_radio_end_backup(void);
                        void lll_prof_cputime_capture(void);
                        void lll_prof_send(void);
                    ```
                7. lll_scan.h
                    ```C
                        int lll_scan_init(void);
                        int lll_scan_reset(void);
                        void lll_scan_prepare(void *param);
                        extern u16_t ull_scan_lll_handle_get(struct lll_scan *lll);
                    ```
                8. lll_slave.h
                    ```C
                        int lll_slave_init(void);
                        int lll_slave_reset(void);
                        void lll_slave_prepare(void *param);
                    ```
                9. lll_tim_internal.h
                    ```C
                        static inline u32_t addr_us_get(u8_t phy);
                    ```
                10. lll_vendor.h
                    ```C
                        //一些宏的定义
                    ```

        * lll-common
            1. lll_conn.h
            2. lll.h            //

* now 的分析
    * lll层的分析
        * platform平台
            * 特殊放在各个平台下的 c 文件
                1. lll_conn.c   //头文件在 lll-common 层
                2. lll.c        //头文件在 lll-common 层
                3. lll_conn_iso.c    //头文件在 lll-common 层
                4. lll_adv.c    //头文件在 lll-common 层
                5. lll_adv_aux.c
                6. lll_adv_iso.c
                7. lll_adv_sync.c
                8. lll_central.c
                9. lll_clock.c
                10. lll_conn_iso.c
                11. lll_df.c
                12. lll_peripheral.c
                13. lll_scan.c
                14. lll_scan_aux.c
                15. lll_sync.c
                16. lll_sync_iso.c
            * 头文件的大概分析
                1. lll_adv_internal.h
                    ```C
                        struct pdu_adv *lll_adv_pdu_latest_get(struct lll_adv_pdu *pdu, u8_t *is_modified);
                        struct pdu_adv *lll_adv_data_latest_get(struct lll_adv *lll, u8_t *is_modified);
                        struct pdu_adv *lll_adv_scan_rsp_latest_get(struct lll_adv *lll,u8_t *is_modified);
                        struct pdu_adv *lll_adv_scan_rsp_curr_get(struct lll_adv *lll);

                        bool lll_adv_scan_req_check(struct lll_adv *lll, struct pdu_adv *sr, uint8_t tx_addr, uint8_t *addr, uint8_t devmatch_ok, uint8_t *rl_idx);
                        
                        bool lll_adv_connect_ind_check(struct lll_adv *lll, struct pdu_adv *ci, uint8_t tx_addr, uint8_t *addr, uint8_t rx_addr, uint8_t *tgt_addr, uint8_t devmatch_ok, uint8_t *rl_idx);
                    ```
                2. lll_adv_pdu.h
                    ```C
                        int lll_adv_data_init(struct lll_adv_pdu *pdu);
                        int lll_adv_data_reset(struct lll_adv_pdu *pdu);
                        int lll_adv_data_dequeue(struct lll_adv_pdu *pdu);
                        int lll_adv_data_release(struct lll_adv_pdu *pdu);
                        void lll_adv_pdu_enqueue(struct lll_adv_pdu *pdu, uint8_t idx);
                        struct pdu_adv *lll_adv_pdu_alloc(struct lll_adv_pdu *pdu, uint8_t *idx);
                        struct pdu_adv *lll_adv_pdu_alloc_pdu_adv(void);
                        struct pdu_adv *lll_adv_data_alloc(struct lll_adv *lll, uint8_t *idx);
                        void lll_adv_data_enqueue(struct lll_adv *lll, uint8_t idx);
                        pdu_adv *lll_adv_data_peek(struct lll_adv *lll);
                        struct pdu_adv *lll_adv_data_curr_get(struct lll_adv *lll);
                        struct pdu_adv *lll_adv_scan_rsp_alloc(struct lll_adv *lll, uint8_t *idx);
                        void lll_adv_scan_rsp_enqueue(struct lll_adv *lll, uint8_t idx);
                        struct pdu_adv *lll_adv_scan_rsp_peek(struct lll_adv *lll);
                        struct pdu_adv *lll_adv_pdu_latest_peek(const struct lll_adv_pdu *const pdu);
                        struct pdu_adv *lll_adv_data_latest_peek(const struct lll_adv *const lll);
                    ```
                3. lll_adv_types.h
                    ```C
                        struct lll_adv_pdu {

                        };
                    ```
                4. lll_df_internal.h
                    ```C
                        void lll_df_cte_tx_enable(struct lll_adv_sync *lll_sync, const struct pdu_adv *pdu,uint32_t *cte_len_us);
                        void lll_df_conf_cte_tx_disable(void);
                        struct lll_df_sync_cfg *lll_df_sync_cfg_alloc(struct lll_df_sync *df_cfg, uint8_t *idx);
                        struct lll_df_sync_cfg *lll_df_sync_cfg_peek(struct lll_df_sync *df_cfg);
                        void lll_df_sync_cfg_enqueue(struct lll_df_sync *df_cfg, uint8_t idx);
                        struct lll_df_sync_cfg *lll_df_sync_cfg_latest_get(struct lll_df_sync *df_cfg,uint8_t *is_modified);
                        struct lll_df_sync_cfg *lll_df_sync_cfg_curr_get(struct lll_df_sync *df_cfg);
                        uint8_t lll_df_sync_cfg_is_modified(struct lll_df_sync *df_cfg);
                        void lll_df_conf_cte_rx_enable(uint8_t slot_duration, uint8_t ant_num, uint8_t *ant_ids, uint8_t chan_idx);
                    ```
                5. lll_df_types.h
                    ```C
                        //一些数据结构的定义
                    ```
                6. lll_internal.h
                    ```C
                        int lll_prepare_done(void *param);
                        int lll_done(void *param);
                        bool lll_is_done(void *param);
                        int lll_is_abort_cb(void *next, void *curr, lll_prepare_cb_t *resume_cb);
                        void lll_abort_cb(struct lll_prepare_param *prepare_param, void *param);
                        uint32_t lll_event_offset_get(struct ull_hdr *ull);
                        uint32_t lll_preempt_calc(struct ull_hdr *ull, uint8_t ticker_id,uint32_t ticks_at_event);
                        void lll_chan_set(uint32_t chan);
                        void lll_isr_tx_status_reset(void);
                        void lll_isr_rx_status_reset(void);
                        void lll_isr_status_reset(void);
                        void lll_isr_abort(void *param);
                        void lll_isr_done(void *param);
                        void lll_isr_cleanup(void *param);
                        void lll_isr_early_abort(void *param);
                    ```
                7. lll_port_internal.h
                    ```C
                        void lll_prof_enter_radio(void);
                        void lll_prof_exit_radio(void);
                        void lll_prof_enter_lll(void);
                        void lll_prof_exit_lll(void);
                        void lll_prof_enter_ull_high(void);
                        void lll_prof_exit_ull_high(void);
                        void lll_prof_enter_ull_low(void);
                        void lll_prof_exit_ull_low(void);

                        void lll_prof_latency_capture(void);
                        void lll_prof_radio_end_backup(void);
                        void lll_prof_cputime_capture(void);
                        void lll_prof_send(void);
                        struct node_rx_pdu *lll_prof_reserve(void);
                        void lll_prof_reserve_send(struct node_rx_pdu *rx);
                    ```
                8. lll_scan_internal.h
                    ```C
                        void lll_scan_isr_resume(void *param);
                        bool lll_scan_isr_rx_check(const struct lll_scan *lll, uint8_t irkmatch_ok,uint8_t devmatch_ok, uint8_t rl_idx);
                        bool lll_scan_adva_check(const struct lll_scan *lll, uint8_t addr_type,const uint8_t *addr, uint8_t rl_idx);
                        bool lll_scan_ext_tgta_check(const struct lll_scan *lll, bool pri, bool is_init,const struct pdu_adv *pdu, uint8_t rl_idx,bool *const dir_report);
                        void lll_scan_prepare_connect_req(struct lll_scan *lll, struct pdu_adv *pdu_tx,uint8_t phy, uint8_t adv_tx_addr,uint8_t *adv_addr, uint8_t init_tx_addr,uint8_t *init_addr, uint32_t *conn_space_us);
                        uint8_t lll_scan_aux_setup(struct pdu_adv *pdu, uint8_t pdu_phy,uint8_t pdu_phy_flags_rx, radio_isr_cb_t setup_cb,void *param);
                        void lll_scan_aux_isr_aux_setup(void *param);
                        bool lll_scan_aux_addr_match_get(const struct lll_scan *lll,const struct pdu_adv *pdu,uint8_t *const devmatch_ok,uint8_t *const devmatch_id,uint8_t *const irkmatch_ok,uint8_t *const irkmatch_id);
                    ```
                9. lll_sync_internal.h
                    ```C
                        void lll_sync_aux_prepare_cb(struct lll_sync *lll,struct lll_scan_aux *lll_aux);
                    ```
                10. lll_tim_internal.h
                    ```C
                        uint32_t addr_us_get(uint8_t phy);
                    ```
                11. lll_vendor.h
                    ```C
                        //一些宏的定义
                    ```
        * lll-common
    * 
