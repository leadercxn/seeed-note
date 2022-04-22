# Greybus


## terminology 术语
* Gbridge
    

## greybus for zephyr 
* 源码路径
    1. greybus-for-zephyr-mikrobus/subsys/greybus/

* 编译注意
    1. export ZEPHYR_EXTRA_MODULES=${ZEPHYR_EXTRA_MODULES:-$ZPRJ/greybus-for-zephyr-mikrobus} 要指向需要编译的目录

* 数据结构
    1. greybus.h 下的
        ```C
            struct gb_transport_backend {
                void (*init)(void);
                void (*exit)(void);
                int (*listen)(unsigned int cport);
                int (*stop_listening)(unsigned int cport);
                int (*send)(unsigned int cport, const void *buf, size_t len);
                int (*send_async)(unsigned int cportid, const void *buf, size_t len,
                                unipro_send_completion_t callback, void *priv);
                void *(*alloc_buf)(size_t size);
                void (*free_buf)(void *ptr);
            };
        ```

* 源码分析



* 疑问整理
    1. manifest 是啥意思？
    2. 