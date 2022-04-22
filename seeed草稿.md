* USB烧录命令
STM32_Programmer_CLI -c port=usb1 -w flashlayout_st-image-weston/trusted/FlashLayout_sdcard_stm32mp157c-dk2-trusted.tsv

* distribution包的使用
    + repo init -u https://github.com/STMicroelectronics/oe-manifest.git -b refs/tags/openstlinux-5.10-dunfell-mp1-21-03-31
    + repo sync

    + DISTRO=openstlinux-weston MACHINE=stm32mp1 source layers/meta-st/scripts/envsetup.sh

    + DISTRO=openstlinux-weston MACHINE=stm32mp1-disco source layers/meta-st/scripts/envsetup.sh

    bitbake openstlinux-weston

* 打补丁
    for p in `ls -1 ../*.patch`; do patch -p1 < $p; done

# TF-A 分析
## 汇编 `bl2_el3_entrypoint.S`
* bl2_main.c
    1. bl	bl2_el3_setup
        -> bl2_el3_early_platform_setup(arg0, arg1, arg2, arg3);
            -> stm32mp_save_boot_ctx_address(arg0);

        -> bl2_el3_plat_arch_setup();

    2. bl	bl2_main

        -> next_bl_ep_info = bl2_load_images(); //加载信息BL3X镜像
            ->  bl2_plat_handle_pre_image_load(bl2_node_info->image_id);




# ST的分区
#Opt	Id	    Name	    Type	    IP	Offset	        Binary
-	    0x01	fsbl1-boot	Binary	    none	0x0	        arm-trusted-firmware/tf-a-stm32mp157c-dk2-usb.stm32
-	    0x03	fip-boot	Binary	    none	0x0	        fip/fip-stm32mp157c-dk2-trusted.bin

P	    0x04	fsbl1	    Binary	    mmc0	0x00004400	arm-trusted-firmware/tf-a-stm32mp157c-dk2-sdcard.stm32
P	    0x05	fsbl2	    Binary	    mmc0	0x00044400	arm-trusted-firmware/tf-a-stm32mp157c-dk2-sdcard.stm32
PD	    0x06	fip	        Binary	    mmc0	0x00084400	fip/fip-stm32mp157c-dk2-trusted.bin
P	    0x10	boot	    System	    mmc0	0x00484400	st-image-bootfs-openstlinux-weston-stm32mp1.ext4
P	    0x11	vendorfs	FileSystem	mmc0	0x04484400	st-image-vendorfs-openstlinux-weston-stm32mp1.ext4
P	    0x12	rootfs	    FileSystem	mmc0	0x05484400	st-image-weston-openstlinux-weston-stm32mp1.ext4
P	    0x13	userfs	    FileSystem	mmc0	0x33E84400	st-image-userfs-openstlinux-weston-stm32mp1.ext4

sdb      8:16   1  14.9G  0 disk
├─sdb1   8:17   1   256K  0 part
├─sdb2   8:18   1   256K  0 part
├─sdb3   8:19   1     2M  0 part
├─sdb4   8:20   1    64M  0 part /media/cxn/bootfs
├─sdb5   8:21   1    16M  0 part /media/cxn/vendorfs
├─sdb6   8:22   1   746M  0 part /media/cxn/rootfs
└─sdb7   8:23   1    14G  0 part /media/cxn/userfs

# FIP bin文件的信息
fiptool info fip-stm32mp157c-dk2-trusted.bin
Secure Payload BL32 (Trusted OS): offset=0x100, size=0x1347C, cmdline="--tos-fw"
Non-Trusted Firmware BL33: offset=0x1357C, size=0xCF9D8, cmdline="--nt-fw"
FW_CONFIG: offset=0xE2F54, size=0x226, cmdline="--fw-config"
HW_CONFIG: offset=0xE317A, size=0x1CF56, cmdline="--hw-config"
TOS_FW_CONFIG: offset=0x1000D0, size=0x46E4, cmdline="--tos-fw-config"




# seeed分区
sudo sgdisk --resize-table=128 -a 1 \
        -n 1:34:545      -c 1:fsbl1   \
        -n 2:546:1057    -c 2:fsbl2   \
        -n 3:1058:5153   -c 3:ssbl    \
        -n 4:5154:136225 -c 4:boot    \
        -n 5:136226:     -c 5:rootfs  \
        -p ${DISK}

sdb      8:16   1   7.5G  0 disk
├─sdb1   8:17   1   256K  0 part
├─sdb2   8:18   1   256K  0 part
├─sdb3   8:19   1     2M  0 part
├─sdb4   8:20   1    64M  0 part /media/cxn/BOOT
└─sdb5   8:21   1   7.4G  0 part /media/cxn/rootfs




io_dev_info_t *dev = handle->dev_handle;
    result = dev->funcs->size(entity, length);


# 日志打印
[%h-%m-%s.%t]


# fip 固件打包命令
fiptool create --fw-config fw-config.dtb \
          --hw-config u-boot.dtb \
          --tos-fw-config bl32.dtb \
          --tos-fw bl32.bin \ 
          --nt-fw u-boot-nodtb.bin \
          fip.bin

fiptool create --fw-config  stm32mp157c-seeed-npi-fw-config-trusted.dtb \
          --hw-config  u-boot-stm32mp157c-dk2-trusted.dtb \
          --tos-fw-config  stm32mp157c-seeed-npi-bl32.dtb \
          --tos-fw  tf-a-bl32-stm32mp15.bin  \
          --nt-fw  u-boot-nodtb-stm32mp15.bin \
          seeed-use-dk2-fip.bin

fiptool update --tos-fw <tfa_path>/bl32.bin fip.bin


# u-boot
* printenv
[17-09-40.641]fdt_addr_r=0xc4000000
[17-09-40.641]fdtcontroladdr=dbdea440
[17-09-40.641]kernel_addr_r=0xc2000000

fatload <interface> [<dev[:part]> [<addr> [<filename> [bytes [pos]]]]]


fatload mmc 0:4 0xc2000000 uImage
fatload mmc 0:4 0xc4000000 stm32mp157c-odyssey.dtb
setenv bootargs 'console=ttySTM0,115200 root=/dev/mmcblk0p5 rootfstype=ext4 rootwait rw'
bootm 0xc2000000 - 0xc4000000
或
fatload mmc 0:4 0xc2000000 zImage
bootz 0xc2000000 - 0xc4000000

ext4load mmc 1:2 C2000000 uImage


/dtbs/5.5.19-armv7-lpae-x26/stm32mp157c-seeed-npi.dtb
/vmlinuz-5.5.19-armv7-lpae-x26

make -f ../Makefile.sdk all UBOOT_CONFIGS=../basic/.config,basic,u-boot.img DEVICE_TREE=stm32mp157c-odyssey


# seeed-odyseey 设备树
&ethernet0 {
	status = "okay";
	pinctrl-0 = <&ethernet0_rgmii_pins_a>;
	pinctrl-1 = <&ethernet0_rgmii_sleep_pins_a>;
	pinctrl-names = "default", "sleep";
	phy-mode = "rgmii-id";
	max-speed = <1000>;
	phy-handle = <&phy0>;

	mdio0 {
		#address-cells = <1>;
		#size-cells = <0>;
		compatible = "snps,dwmac-mdio";
		phy0: ethernet-phy@0 {
			reg = <0>;
		};
	};
}; 


# ST设备树
&ethernet0 {
	status = "okay";
	pinctrl-0 = <&ethernet0_rgmii_pins_a>;
	pinctrl-1 = <&ethernet0_rgmii_sleep_pins_a>;
	pinctrl-names = "default", "sleep";
	phy-mode = "rgmii-id";
	max-speed = <1000>;
	phy-handle = <&phy0>;
	nvmem-cells = <&ethernet_mac_address>;
	nvmem-cell-names = "mac-address";

	mdio0 {
		#address-cells = <1>;
		#size-cells = <0>;
		compatible = "snps,dwmac-mdio";
		phy0: ethernet-phy@0 {
			reg = <0>;
		};
	};
};


data = 
0000000 0527 5619 93d6 9d89 ef60 eeb7 7100 50cd
0000010 00c2 4000 00c2 4000 a6fc b7c7 0205 0002
0000020 694c 756e 2d78 2e35

load: C200 0040
len= 7458128
dcrc = B239 99AD = 2990119341
image_dcrc = a6fc b7c7 = 4238788535


# npi-common.h
```C
/*
 *  test -e 文件是否存在
 *  test -n 字符串不为空
 *       -z 字符串为空
 *       -o  或
 *       -a  与
*/

#define NPI_BOOT \
	"boot=mmc dev 0; " \
		"if mmc rescan; then " \
			"setenv bootpart 0:1; " \
			"if test -e mmc 0:1 /etc/fstab; then " \        # 0:1无这个文件
				"setenv mmcpart 1;" \
			"fi; " \
			"echo Checking for: /uEnv.txt ...;" \
			"if test -e mmc 0:1 /uEnv.txt; then " \         # 0:1无这个文件
				"if run loadbootenv; then " \               # load mmc 0:1 0xC2000000 uEnv.txt
					"echo Loaded environment from /uEnv.txt;" \
					"run importbootenv;" \      # echo Importing environment from mmc ...; env import -t 0xC2000000 ${filesize}
				"fi;" \
				"echo Checking if uenvcmd is set ...;" \
				"if test -n ${uenvcmd}; then " \            # 无 uenvcmd
					"echo Running uenvcmd ...;" \
					"run uenvcmd; setenv uenvcmd;" \
				"fi;" \
				"echo Checking if client_ip is set ...;" \
				"if test -n ${client_ip}; then " \          # 无 client_ip
					"if test -n ${dtb}; then " \
						"setenv fdtfile ${dtb};" \
						"echo using ${fdtfile} ...;" \
					"fi;" \
					"if test -n ${uname_r}; then " \
						"echo Running nfsboot_uname_r ...;" \
						"run nfsboot_uname_r;" \
					"fi;" \
					"echo Running nfsboot ...;" \
					"run nfsboot;" \
				"fi;" \
			"fi; " \
			"echo Checking for: boot.scr ...;" \
			"if test -e mmc 0:1 /boot.scr; then " \     # 无boot.scr文件
				"setenv scriptfile ${script};" \
				"run loadbootscript;" \
				"echo Loaded script from ${scriptfile};" \
				"run bootscript;" \
			"fi; " \
			"echo Checking for: /boot/boot.scr ...;" \
			"if test -e mmc 0:1 /boot/boot.scr ; then " \   # 无boot.scr文件
				"setenv scriptfile /boot/${script};" \
				"run loadbootscript;" \
				"echo Loaded script from ${scriptfile};" \
				"run bootscript;" \
			"fi; " \
			"echo Checking for: /boot/uEnv.txt ...;" \
			"for i in 1 2 3 4 5 6 7 ; do " \            # 便利各个分区去查找 uEnv.txt文件
				"setenv mmcpart ${i};" \
				"setenv curpart 0:${mmcpart};" \
				"if test -e mmc 0:${mmcpart} /uEnv.txt; then " \    # 假如在某一个分区找到 uEnv.txt 文件
					"setenv bootpart 0:${mmcpart};" \       # bootpart = 找到放了uImage的分区号
					"load mmc 0:4 0xC2000000 /uEnv.txt;" \
					"env import -t 0xC2000000 ${filesize};" \
					"echo Loaded environment from /uEnv.txt;" \
					"if test -n ${uenvcmd}; then " \            # 无 uenvcmd
						"echo Running uenvcmd ...${uenvcmd};" \
						"run uenvcmd; setenv uenvcmd;" \
					"fi;" \
				"fi;" \
				"if test -e mmc 0:6 /bin/sh; then " \               # 文件系统
					"setenv rootpart 0:6;" \                # 找到放了文件系统的分区号
					"if test -n ${dtb}; then " \            # uEnv.txt中 dtb 文件名 dtb=stm32mp1-seeed-npi-base.dtb
						"echo debug: [dtb=${dtb}] ... ;" \
						"setenv fdtfile ${dtb};" \          # fdtfile = stm32mp1-seeed-npi-base.dtb
						"echo Using: dtb=${fdtfile} ...;" \
					"fi;" \
					"echo Checking if uname_r is set in /uEnv.txt...;" \ #在uEnv.txt中 uname_r = 4.19.9-stm32-r1
					"if test -n ${uname_r}; then " \
						"setenv oldroot /dev/mmcblk0p6;" \ # 指向文件系统的位置
						"echo Running uname_boot ...;" \
						"run uname_boot;" \
					"fi;" \
				"fi;" \
			"done;" \
		"fi;\0" \
\
    "uname_boot="\
		"setenv bootdir /boot; " \
		"setenv bootfile vmlinuz-4.19.9-stm32-r1; " \
		"if test -e mmc 0:4 /boot/vmlinuz-4.19.9-stm32-r1; then " \
			"true;" \
		"else " \
			"setenv bootdir ;" \
		"fi;" \
		"if test -e mmc 0:4 /vmlinuz-4.19.9-stm32-r1; then " \  # bootdir 为空
			"echo loading /vmlinuz-4.19.9-stm32-r1 ...; "\
			"run loadimage;" \                                  #load mmc 0:40xC2000000 /vmlinuz-4.19.9-stm32-r1        # 加载uImage到DRAM
			"setenv fdtdir /dtbs/4.19.9-stm32-r1; " \
			"echo debug: [enable_uboot_overlays=1] ... ;" \ # 在uEnv.txt文件中 enable_uboot_overlays=1
			"if test -n 1; then " \
				"echo debug: [uboot_base_dtb=${uboot_base_dtb}] ... ;" \    uboot_base_dtb 为空
				"if test -n ; then " \
					"echo uboot_overlays: [uboot_base_dtb=${uboot_base_dtb}] ... ;" \
					"if test -e mmc 0:4 /dtbs/4.19.9-stm32-r1/${uboot_base_dtb}; then " \
						"setenv fdtfile ${uboot_base_dtb};" \
						"echo uboot_overlays: Switching to: dtb=${fdtfile} ...;" \
					"fi;" \
				"fi;" \
			"fi;" \
			"if test -e mmc 0:4 /dtbs/4.19.9-stm32-r1/stm32mp1-seeed-npi-base.dtb; then " \ # 加载设备树文件到DRAM
				"run loadfdt;" \  #echo loading /dtbs/4.19.9-stm32-r1/stm32mp1-seeed-npi-base.dtb ...; load mmc 0:4 $0xC4000000 /dtbs/4.19.9-stm32-r1/stm32mp1-seeed-npi-base.dtb
			"else " \
				"setenv fdtdir /usr/lib/linux-image-4.19.9-stm32-r1; " \    # 设备树在其他路径，暂不用理
				"if test -e mmc 0:4 ${fdtdir}/${fdtfile}; then " \
					"run loadfdt;" \
				"else " \
					"setenv fdtdir /boot/dtbs; " \
					"if test -e ${devtype} ${bootpart} ${fdtdir}/${fdtfile}; then " \
						"run loadfdt;" \
					"else " \
						"setenv fdtdir /boot; " \
						"if test -e ${devtype} ${bootpart} ${fdtdir}/${fdtfile}; then " \
							"run loadfdt;" \
						"else " \
							"if test -e ${devtype} ${bootpart} ${fdtfile}; then " \
								"run loadfdt;" \
							"else " \
								"echo; echo unable to find [dtb=${fdtfile}] did you name it correctly? ...; " \
								"run failumsboot;" \
							"fi;" \
						"fi;" \
					"fi;" \
				"fi;" \
			"fi; " \
			"if test -n ${enable_uboot_overlays}; then " \  # enable_uboot_overlays = 1
				"setenv fdt_buffer 0x60000;" \
				"if test -n ${uboot_fdt_buffer}; then " \   # 在uEnv.txt文件中 #uboot_fdt_buffer=0x60000，即不用理
					"setenv fdt_buffer ${uboot_fdt_buffer};" \
				"fi;" \
				"echo uboot_overlays: [fdt_buffer=${fdt_buffer}] ... ;" \
				"if test -n ${uboot_silicon}; then " \      # 空
					"setenv uboot_overlay ${uboot_silicon}; " \
					"run virtualloadoverlay;" \
				"fi;" \
				"if test -n ${uboot_model}; then " \        # 空
					"setenv uboot_overlay ${uboot_model}; " \
					"run virtualloadoverlay;" \
				"fi;" \
				"if test -n ${uboot_overlay_addr0}; then " \  空  uEnv.txt文件中 #uboot_overlay_addr0=/lib/firmware/<file0>.dtbo
					"setenv uboot_overlay ${uboot_overlay_addr0}; " \
					"run capeloadoverlay;" \
				"fi;" \
				"if test -n ${uboot_overlay_addr1}; then " \
					"setenv uboot_overlay ${uboot_overlay_addr1}; " \
					"run capeloadoverlay;" \
				"fi;" \
				"if test -n ${uboot_overlay_addr2}; then " \
					"setenv uboot_overlay ${uboot_overlay_addr2}; " \
					"run capeloadoverlay;" \
				"fi;" \
				"if test -n ${uboot_overlay_addr3}; then " \
					"setenv uboot_overlay ${uboot_overlay_addr3}; " \
					"run capeloadoverlay;" \
				"fi;" \
				"if test -n ${uboot_overlay_addr4}; then " \
					"setenv uboot_overlay ${uboot_overlay_addr4}; " \
					"run capeloadoverlay;" \
				"fi;" \
				"if test -n ${uboot_overlay_addr5}; then " \
					"setenv uboot_overlay ${uboot_overlay_addr5}; " \
					"run capeloadoverlay;" \
				"fi;" \
				"if test -n ${uboot_overlay_addr6}; then " \
					"setenv uboot_overlay ${uboot_overlay_addr6}; " \
					"run capeloadoverlay;" \
				"fi;" \
				"if test -n ${uboot_overlay_addr7}; then " \
					"setenv uboot_overlay ${uboot_overlay_addr7}; " \
					"run capeloadoverlay;" \
				"fi;" \
			"else " \  # 空
				"echo uboot_overlays: add [enable_uboot_overlays=1] to /boot/uEnv.txt to enable...;" \
			"fi;" \
			"setenv rdfile initrd.img-4.19.9-stm32-r1; " \
			"if test -e mmc 0:4 /initrd.img-4.19.9-stm32-r1; then " \ 真
				"echo loading /initrd.img-4.19.9-stm32-r1 ...; "\
				"run loadrd;" \ # load mmc 0:4 0xC4400000 /initrd.img-4.19.9-stm32-r1; setenv rdsize ${filesize}    # 加载 initrd.img 到DRAM
				"if test -n ${netinstall_enable}; then " \  # 空
					"run args_netinstall; run message;" \
					"echo debug: [${bootargs}] ... ;" \
					"echo debug: [bootz ${loadaddr} ${rdaddr}:${rdsize} ${fdtaddr}] ... ;" \
					"bootz ${loadaddr} ${rdaddr}:${rdsize} ${fdtaddr}; " \
				"fi;" \
				"if test -n ${uenv_root}; then " \  # 可能为空
					"run args_uenv_root;" \
					"echo debug: [${bootargs}] ... ;" \
					"echo debug: [bootz ${loadaddr} ${rdaddr}:${rdsize} ${fdtaddr}] ... ;" \
					"bootz ${loadaddr} ${rdaddr}:${rdsize} ${fdtaddr}; " \
				"fi;" \
				"if test -n ${uuid}; then " \   #可能为空   uEnv.txt中 #uuid=
					"run args_mmc_uuid;" \
					"echo debug: [${bootargs}] ... ;" \
					"echo debug: [bootz ${loadaddr} ${rdaddr}:${rdsize} ${fdtaddr}] ... ;" \
					"bootz ${loadaddr} ${rdaddr}:${rdsize} ${fdtaddr}; " \
				"fi;" \
                \ # uEnv.txt中的cmdline= coherent_pool=1M net.ifnames=0 quiet   # coherent_pool:DMA内存池的大小 net.ifnames=0 禁用网卡命名规则 quiet 在内核启动的时候不打印日志
				"run args_mmc_old;" \  # setenv bootargs console=${console} ${optargs} root=/dev/mmcblk0p6 ro rootfstype=ext4 rootwait ${cmdline}
				"echo debug: [${bootargs}] ... ;" \
				"echo debug: [bootz ${loadaddr} ${rdaddr}:${rdsize} ${fdtaddr}] ... ;" \
				"bootz ${loadaddr} ${rdaddr}:${rdsize} ${fdtaddr}; " \
			"else " \   # 空
				"if test -n ${uenv_root}; then " \
					"run args_uenv_root;" \
					"echo debug: [${bootargs}] ... ;" \
					"echo debug: [bootz ${loadaddr} - ${fdtaddr}] ... ;" \
					"bootz ${loadaddr} - ${fdtaddr}; " \
				"fi;" \
                \ 
				"run args_mmc_old;" \
				"echo debug: [${bootargs}] ... ;" \
				"echo debug: [bootz ${loadaddr} - ${fdtaddr}] ... ;" \
				"bootz ${loadaddr} - ${fdtaddr}; " \
			"fi;" \
		"else " \
			"echo debug: file missing ${devtype} ${bootpart} ${bootdir}/${bootfile};" \
		"fi;\0" \
```



bitbake -e tf-a-stm32mp | grep ^TF_A_DEVICETREE=
TF_A_DEVICETREE="   
stm32mp157c-odyssey   stm32mp157a-dk1 stm32mp157d-dk1 stm32mp157c-dk2 stm32mp157f-dk2   stm32mp157c-ed1 stm32mp157f-ed1   stm32mp157a-ev1 stm32mp157c-ev1 stm32mp157d-ev1 stm32mp157f-ev1 ,   
stm32mp157c-odyssey   stm32mp157a-dk1 stm32mp157d-dk1 stm32mp157c-dk2 stm32mp157f-dk2   stm32mp157c-ed1 stm32mp157f-ed1   stm32mp157a-ev1 stm32mp157c-ev1 stm32mp157d-ev1 stm32mp157f-ev1 ,  
stm32mp157c-ed1 stm32mp157f-ed1  stm32mp157a-ev1 stm32mp157c-ev1 stm32mp157d-ev1 stm32mp157f-ev1,  
stm32mp157c-ed1 stm32mp157f-ed1  stm32mp157a-ev1 stm32mp157c-ev1 stm32mp157d-ev1 stm32mp157f-ev1,  
stm32mp157a-ev1 stm32mp157c-ev1 stm32mp157d-ev1 stm32mp157f-ev1,  
stm32mp157a-dk1 stm32mp157d-dk1 stm32mp157c-dk2 stm32mp157f-dk2  stm32mp157c-ed1 stm32mp157f-ed1  stm32mp157a-ev1 stm32mp157c-ev1 stm32mp157d-ev1 stm32mp157f-ev1, 
stm32mp157c-odyssey   stm32mp157a-dk1 stm32mp157d-dk1 stm32mp157c-dk2 stm32mp157f-dk2   stm32mp157c-ed1 stm32mp157f-ed1   stm32mp157a-ev1 stm32mp157c-ev1 stm32mp157d-ev1 stm32mp157f-ev1 ,   
stm32mp157c-odyssey   stm32mp157a-dk1 stm32mp157d-dk1 stm32mp157c-dk2 stm32mp157f-dk2   stm32mp157c-ed1 stm32mp157f-ed1   stm32mp157a-ev1 stm32mp157c-ev1 stm32mp157d-ev1 stm32mp157f-ev1 ,"

bitbake -e tf-a-stm32mp | grep ^TF_A_CONFIG=
TF_A_CONFIG=" optee trusted  emmc nand nor sdcard  uart usb "

bitbake -e tf-a-stm32mp | grep ^TF_A_EXTRA_OPTFLAGS=
TF_A_EXTRA_OPTFLAGS="AARCH32_SP=optee ,AARCH32_SP=sp_min ,STM32MP_EMMC=1,STM32MP_RAW_NAND=1 STM32MP_FORCE_MTD_START_OFFSET=0x00200000,STM32MP_SPI_NOR=1 STM32MP_FORCE_MTD_START_OFFSET=0x00080000,STM32MP_SDMMC=1,STM32MP_UART_PROGRAMMER=1,STM32MP_USB_PROGRAMMER=1,"

bitbake -e tf-a-stm32mp | grep ^TF_A_BINARIES=
TF_A_BINARIES="tf-a,tf-a,tf-a,tf-a,tf-a,tf-a,tf-a,tf-a,"

bitbake -e tf-a-stm32mp | grep ^TF_A_MAKE_TARGET=
TF_A_MAKE_TARGET="dtbs,bl32 dtbs,all,all,all,all,all,all,"


* 原st
AARCH32_SP=optee  
AARCH32_SP=sp_min 
STM32MP_EMMC=1
STM32MP_RAW_NAND=1 STM32MP_FORCE_MTD_START_OFFSET=0x00200000
STM32MP_SPI_NOR=1 STM32MP_FORCE_MTD_START_OFFSET=0x00080000
STM32MP_SDMMC=1
STM32MP_UART_PROGRAMMER=1
STM32MP_USB_PROGRAMMER=1

tf-a * 8




make ARCH=arm uImage vmlinux dtbs LOADADDR=0xC2000040
make ARCH=arm modules

lcd@45
port {
	endpoint {
			remote-endpoint = <0x18>;
			phandle = <0x3c>;
	};
}

dsi
port@1 {
        reg = <0x01>;
        endpoint {
                remote-endpoint = <0x3c>;
                phandle = <0x18>;
        };
};

port@0 {
        reg = <0x00>;
        endpoint {
                remote-endpoint = <0x3b>;
                phandle = <0x35>;
        };
};

ltdc
port {
        #address-cells = <0x01>;
        #size-cells = <0x00>;
        endpoint@1 {
                remote-endpoint = <0x35>;
                phandle = <0x3b>;
                reg = <0x01>;
        };
};




* seeed
Encoders:
id      crtc    type    possible crtcs  possible clones
31      35      DSI     0x00000001      0x00000001

Connectors:
id      encoder status          name            size (mm)       modes   encoders
32      31      connected       DSI-1           154x86          2       31
connector->32
encoder  ->31
crtc     ->35



* ST:
connector->29
encoder  ->28
crtc     ->33

* vc4
connector->42
encoder  ->41
crtc     ->69

* modetest
-P <plane_id>@<crtc_id>:<w>x<h>[+<x>+<y>][*<scale>][@<format>]  set a plane
-s <connector_id>[,<connector_id>][@<crtc_id>]:<mode>[-<vrefresh>][@<format>]   set a mode

-d      drop master after mode set
-M module       use the given driver
-D device       use the given device


######
/* General DSI hardware state. */
struct vc4_dsi {
	struct platform_device *pdev;

	struct mipi_dsi_host dsi_host;
	struct drm_encoder *encoder;
	struct drm_bridge *bridge;
	struct list_head bridge_chain;

	void __iomem *regs;

	struct dma_chan *reg_dma_chan;
	dma_addr_t reg_dma_paddr;
	u32 *reg_dma_mem;
	dma_addr_t reg_paddr;

	const struct vc4_dsi_variant *variant;

	/* DSI channel for the panel we're connected to. */
	u32 channel;
	u32 lanes;
	u32 format;
	u32 divider;
	u32 mode_flags;

	/* Input clock from CPRMAN to the digital PHY, for the DSI
	 * escape clock.
	 */
	struct clk *escape_clock;

	/* Input clock to the analog PHY, used to generate the DSI bit
	 * clock.
	 */
	struct clk *pll_phy_clock;

	/* HS Clocks generated within the DSI analog PHY. */
	struct clk_fixed_factor phy_clocks[3];

	struct clk_hw_onecell_data *clk_onecell;

	/* Pixel clock output to the pixelvalve, generated from the HS
	 * clock.
	 */
	struct clk *pixel_clock;

	struct completion xfer_completion;
	int xfer_result;

	struct debugfs_regset32 regset;
};



mdio_bus stmmac-0: MDIO device at address 7 is missing.

stm32-dwmac 5800a000.ethernet eth0: no phy at addr -1
stm32-dwmac 5800a000.ethernet eth0: stmmac_open: Cannot attach to PHY (error: -19)


&ethernet0 {
	status = "okay";
	pinctrl-0 = <&ethernet0_rgmii_pins_a>;
	pinctrl-1 = <&ethernet0_rgmii_sleep_pins_a>;
	pinctrl-names = "default", "sleep";
	phy-mode = "rgmii-id";
	max-speed = <1000>;
	phy-handle = <&phy0>;
	assigned-clocks = <&rcc ETHCK_K>;
	assigned-clock-parents = <&rcc PLL4_P>;
	assigned-clock-rates = <125000000>; /* Clock PLL4 to 750Mhz in ATF/U-Boot */
	st,eth-clk-sel;

	mdio0 {
		#address-cells = <1>;
		#size-cells = <0>;
		compatible = "snps,dwmac-mdio";

		phy0: ethernet-phy@7 {	/* KSZ9031RNXCA */
			reg = <7>;

			/*
			 * These skews assume the STM32MP1 has no internal delay.
			 *
			 * All skews are offset since hardware skew values for the ksz9031
			 * range from a negative skew to a positive skew.
			 * See the micrel-ksz90x1.txt Documentation file for details.
			 */
			reset-assert-us = <10000>;
			reset-deassert-us = <300>;

			/* REG 0x0008, 5 bits per skew */
			txc-skew-ps  = <1800>;/*  900ps */
			rxc-skew-ps  = <1320>;/*  420ps */

			/* REG 0x0004, 4 bits per skew */
			txen-skew-ps = <420>;  /*   0ps */
			rxdv-skew-ps = <420>; /*    0ps */

			/* REG 0x0005, 4 bits per skew */
			rxd0-skew-ps = <720>; /*  300ps */
			rxd1-skew-ps = <780>; /*  360ps */
			rxd2-skew-ps = <840>; /*  420ps */
			rxd3-skew-ps = <900>; /*  480ps */

			/* REG 0x0006, 4 bits per skew */
			txd0-skew-ps = <0>;   /* -420ps */
			txd1-skew-ps = <60>;  /* -360ps */
			txd2-skew-ps = <120>; /* -300ps */
			txd3-skew-ps = <180>; /* -240ps */

			test-version = <0x02>;

			micrel,force-master;
		};
	};
};

-- phy-mode: string, operation mode of the PHY interface; supported values are
-  "mii", "gmii", "sgmii", "qsgmii", "tbi", "rev-mii", "rmii", "rgmii", "rgmii-id",
-  "rgmii-rxid", "rgmii-txid", "rtbi", "smii", "xgmii", "trgmii"; this is now a
-  de-facto standard property;
+- phy-mode: string, operation mode of the PHY interface. This is now a de-facto
+  standard property; supported values are:
+  * "mii"
+  * "gmii"
+  * "sgmii"
+  * "qsgmii"
+  * "tbi"
+  * "rev-mii"
+  * "rmii"
+  * "rgmii" (RX and TX delays are added by the MAC when required)
+  * "rgmii-id" (RGMII with internal RX and TX delays provided by the PHY, the
+     MAC should not add the RX or TX delays in this case)
+  * "rgmii-rxid" (RGMII with internal RX delay provided by the PHY, the MAC
+     should not add an RX delay in this case)
+  * "rgmii-txid" (RGMII with internal TX delay provided by the PHY, the MAC
+     should not add an TX delay in this case)
+  * "rtbi"
+  * "smii"
+  * "xgmii"


ethernet0: ethernet@5800a000 {
			compatible = "st,stm32mp1-dwmac", "snps,dwmac-4.20a";
			reg = <0x5800a000 0x2000>;
			reg-names = "stmmaceth";
			interrupts-extended = <&intc GIC_SPI 61 IRQ_TYPE_LEVEL_HIGH>,
					      <&exti 70 IRQ_TYPE_LEVEL_HIGH>;
			interrupt-names = "macirq",
					  "eth_wake_irq";
			clock-names = "stmmaceth",
				      "mac-clk-tx",
				      "mac-clk-rx",
				      "eth-ck",
				      "ethstp";
			clocks = <&rcc ETHMAC>,
				 <&rcc ETHTX>,
				 <&rcc ETHRX>,
				 <&rcc ETHCK_K>,
				 <&rcc ETHSTP>;
			st,syscon = <&syscfg 0x4>;
			snps,mixed-burst;
			snps,pbl = <2>;
			snps,en-tx-lpi-clockgating;
			snps,axi-config = <&stmmac_axi_config_0>;
			snps,tso;
			power-domains = <&pd_core>;
			status = "disabled";
		};


scp -r install_artifact/lib/modules/* cxn@192.168.4.101:/home/cxn/seeed/ODYSSEY-NPi/image-mipi/yocto


STM32MP> setenv ipaddr 192.168.4.200
STM32MP> setenv ethaddr 00:04:9f:04:d2:35
STM32MP> setenv gatewayip 192.168.4.1
STM32MP> setenv netmask 255.255.255.0
STM32MP> setenv serverip 192.168.4.101



make all_rpi

sudo dtoverlay   overlays/rpi/ti-cc1120-overlay.dtbo
sudo dtoverlay  -r ti-cc1120-overlay
 
sudo modprobe mac802154

sudo rmmod mac802154

sudo insmod modules/mac802154/mac802154.ko 

sudo insmod modules/cc1120/cc1120.ko
sudo rmmod cc1120

cat /sys/devices/platform/soc/fe204000.spi/spi_master/spi0/spi0.0/cc1120_debug/test 
cat /sys/devices/platform/soc/fe204000.spi/spi_master/spi0/spi0.0/cc1120_debug/tx 
dmesg|tail

scp -r seeed-linux-dtoverlays/ pi@192.168.4.66:/home/pi/dtoverlays

sudo ip link add link wpan0 name lowpan0 type lowpan
sudo ip link set wpan0 up
sudo ip link set lowpan0 up
ip a show lowpan0
ping6 -I lowpan0 ff02::1
sudo ip link set lowpan0 down
sudo ip link set wpan0 down

10.0.0.223
se.101 , qqqqqqqq9
ssh pi@10.0.0.223 raspberry

cd /boot/overlays
sudo dtoverlay spi0-1cs.dtbo 
./spidev_test -D /dev/spidev0.0 -v -p  \\x80\\xff
./spidev_test -D /dev/spidev0.0 -v -p  \\x00\\x46

sudo modprobe -r mac802154

sysfs_create_group


问题点：
1. pinctrl-0 = <&cc1120_int_pins>,
			<&cc1120_gpio_pins>;

2. devm_gpiod_get_index_optional(&spi->dev, "reset", 0,
                                               GPIOD_OUT_LOW);


rx_fifo empty to read | rx_fifo full to write  is error
tx_fifo full to write | tx_fifo empty in the middle of a packet is error


/**
 * enum nl802154_cca_modes - cca modes
 *
 * @__NL802154_CCA_INVALID: cca mode number 0 is reserved
 * @NL802154_CCA_ENERGY: Energy above threshold
 * @NL802154_CCA_CARRIER: Carrier sense only
 * @NL802154_CCA_ENERGY_CARRIER: Carrier sense with energy above threshold
 * @NL802154_CCA_ALOHA: CCA shall always report an idle medium
 * @NL802154_CCA_UWB_SHR: UWB preamble sense based on the SHR of a frame
 * @NL802154_CCA_UWB_MULTIPLEXED: UWB preamble sense based on the packet with
 *	the multiplexed preamble
 * @__NL802154_CCA_ATTR_AFTER_LAST: Internal
 * @NL802154_CCA_ATTR_MAX: Maximum CCA attribute number
 */
enum nl802154_cca_modes {
	__NL802154_CCA_INVALID,
	NL802154_CCA_ENERGY,
	NL802154_CCA_CARRIER,
	NL802154_CCA_ENERGY_CARRIER,
	NL802154_CCA_ALOHA,
	NL802154_CCA_UWB_SHR,
	NL802154_CCA_UWB_MULTIPLEXED,

	/* keep last */
	__NL802154_CCA_ATTR_AFTER_LAST,
	NL802154_CCA_ATTR_MAX = __NL802154_CCA_ATTR_AFTER_LAST - 1
};

/**
 * enum nl802154_cca_opts - additional options for cca modes
 *
 * @NL802154_CCA_OPT_ENERGY_CARRIER_OR: NL802154_CCA_ENERGY_CARRIER with OR
 * @NL802154_CCA_OPT_ENERGY_CARRIER_AND: NL802154_CCA_ENERGY_CARRIER with AND
 */
enum nl802154_cca_opts {
	NL802154_CCA_OPT_ENERGY_CARRIER_AND,
	NL802154_CCA_OPT_ENERGY_CARRIER_OR,

	/* keep last */
	__NL802154_CCA_OPT_ATTR_AFTER_LAST,
	NL802154_CCA_OPT_ATTR_MAX = __NL802154_CCA_OPT_ATTR_AFTER_LAST - 1
};


## 802.15.4/
* 名次的理解
	+ promiscuous mode:	混杂模式又叫偷听模式，指一台机器的网卡能够接收所有经过它的数据流，而不论其目的地址是否是它。


## 
~/github/beagleconnect/sw (master) $ ./flash-cc1352-sensortest.sh 
#!/bin/bash -ve
export PORT=${1:-/dev/tty.usbmodem141301}
export PROJECT=${2:-build/sensortest_beagleconnect}
#export PROJECT=${2:-build/sensortest_beagleconnect_2G}
export SWDIR="$( cd "$(dirname "$0")" >/dev/null 2>&1 ; pwd -P )"
 cd "$(dirname "$0")" >/dev/null 2>&1 ; pwd -P 
dirname "$0"
export ZEPHYR_TOOLCHAIN_VARIANT=${ZEPHYR_TOOLCHAIN_VARIANT:-gnuarmemb}
#export ZEPHYR_SDK_INSTALL_DIR=${ZEPHYR_SDK_INSTALL_DIR:-~/zephyr-sdk-0.11.4}
export GNUARMEMB_TOOLCHAIN_PATH=${GNUARMEMB_TOOLCHAIN_PATH:-/Users/chenxiaonian/tool/gcc-arm-none-eabi-10.3-2021.07-mac-10.14.6/gcc-arm-none-eabi-10.3-2021.07}
export ZEPHYR_BASE=${ZEPHYR_BASE:-$SWDIR/zephyrproject/zephyr}



sudo pkill -KILL appstreamcli
wget -P /tmp https://launchpad.net/ubuntu/+archive/primary/+files/appstream_0.9.4-1ubuntu1_amd64.deb https://launchpad.net/ubuntu/+archive/primary/+files/libappstream3_0.9.4-1ubuntu1_amd64.deb
sudo dpkg -i /tmp/appstream_0.9.4-1ubuntu1_amd64.deb /tmp/libappstream3_0.9.4-1ubuntu1_amd64.deb



## beagleconnect 测试命令
1. 烧录430固件
	sudo python2 -m msp430.bslS.hid_1 -e -P usb_uart_bridge.hex
2. 烧录 cc1352p 固件
	python3 cc2538-bsl.py zephyr.bin /dev/ttyACM0

3. 每秒自动测试led，buzzer

4. 测试flash
	flash erase GD25Q16C 0
	flash write GD25Q16C 0 0x1234
	flash read GD25Q16C 0 4
5. 检测i2c设备
	i2c scan I2C_0

6. 按键

7. IO
	gpio conf GPIO_0 18 out
	gpio set GPIO_0 18 1

8. uart
	两个板子tx ，rx交接或 短接同一板子的tx、rx？


# 周记
1. 熟悉 beagleconnect ,zephyr os 相关
2. 调试 ads1115 驱动
3. cc1352 i2c相关调试
4. 帮国庆检测 beagleconnect_freedom 板子，顺便熟悉相关的传感器



###

* 1.3V
[00:00:59.984,558] <inf> ads1115: salver_addr 0x48 get ain0 value 0x0000
[00:01:00.036,865] <inf> ads1115: salver_addr 0x48 get ain1 value 0x047d
[00:01:00.089,172] <inf> ads1115: salver_addr 0x48 get ain2 value 0x0000
[00:01:00.141,510] <inf> ads1115: salver_addr 0x48 get ain3 value 0x0b48


[00:01:44.027,099] <inf> ads1115: salver_addr 0x49 get ain0 value 0x0000
[00:01:44.079,467] <inf> ads1115: salver_addr 0x49 get ain1 value 0x047d
[00:01:44.131,805] <inf> ads1115: salver_addr 0x49 get ain2 value 0x0000
[00:01:44.184,112] <inf> ads1115: salver_addr 0x49 get ain3 value 0x0b4a


[00:02:06.115,081] <inf> ads1115: salver_addr 0x4b get ain0 value 0x0000
[00:02:06.167,388] <inf> ads1115: salver_addr 0x4b get ain1 value 0x047e
[00:02:06.219,696] <inf> ads1115: salver_addr 0x4b get ain2 value 0x0000
[00:02:06.272,155] <inf> ads1115: salver_addr 0x4b get ain3 value 0x0b38

* 2.6V
[00:02:35.341,247] <inf> ads1115: salver_addr 0x4b get ain0 value 0x0000
[00:02:35.393,554] <inf> ads1115: salver_addr 0x4b get ain1 value 0x0902
[00:02:35.445,892] <inf> ads1115: salver_addr 0x4b get ain2 value 0x0000
[00:02:35.498,352] <inf> ads1115: salver_addr 0x4b get ain3 value 0x0b36

[00:03:00.868,194] <inf> ads1115: salver_addr 0x49 get ain0 value 0x0000
[00:03:00.920,562] <inf> ads1115: salver_addr 0x49 get ain1 value 0x0900
[00:03:00.972,900] <inf> ads1115: salver_addr 0x49 get ain2 value 0x0000
[00:03:01.025,207] <inf> ads1115: salver_addr 0x49 get ain3 value 0x0b4d

[00:03:24.550,201] <inf> ads1115: salver_addr 0x48 get ain0 value 0x0000
[00:03:24.602,508] <inf> ads1115: salver_addr 0x48 get ain1 value 0x08ff
[00:03:24.654,815] <inf> ads1115: salver_addr 0x48 get ain2 value 0x0000
[00:03:24.707,122] <inf> ads1115: salver_addr 0x48 get ain3 value 0x0b48

* 3.9V
[00:06:19.466,491] <inf> ads1115: salver_addr 0x48 get ain0 value 0x0000
[00:06:19.518,798] <inf> ads1115: salver_addr 0x48 get ain1 value 0x0d80
[00:06:19.571,105] <inf> ads1115: salver_addr 0x48 get ain2 value 0x0000
[00:06:19.623,413] <inf> ads1115: salver_addr 0x48 get ain3 value 0x0b4c

[00:06:47.810,485] <inf> ads1115: salver_addr 0x49 get ain0 value 0x0000
[00:06:47.862,854] <inf> ads1115: salver_addr 0x49 get ain1 value 0x0d83
[00:06:47.915,191] <inf> ads1115: salver_addr 0x49 get ain2 value 0x0000
[00:06:47.967,498] <inf> ads1115: salver_addr 0x49 get ain3 value 0x0b49

[00:07:01.130,981] <inf> ads1115: salver_addr 0x4b get ain0 value 0x0000
[00:07:01.183,319] <inf> ads1115: salver_addr 0x4b get ain1 value 0x0d84
[00:07:01.235,626] <inf> ads1115: salver_addr 0x4b get ain2 value 0xffff
[00:07:01.288,085] <inf> ads1115: salver_addr 0x4b get ain3 value 0x0b36


* 2V
[00:07:33.338,623] <inf> ads1115: salver_addr 0x4b get ain0 value 0x0000
[00:07:33.390,960] <inf> ads1115: salver_addr 0x4b get ain1 value 0x06ec
[00:07:33.443,267] <inf> ads1115: salver_addr 0x4b get ain2 value 0x0000
[00:07:33.495,727] <inf> ads1115: salver_addr 0x4b get ain3 value 0x0b36



[00:00:05.891,448] <inf> i2c_cc13xx_cc26xx: I2CMaster base = 0x40002000 




/*z*/
*** Booting Zephyr OS build v2.6.0-rc1-ncs1  ***Flash regionsDomainPermissions00 01 0x00000 0x10000 Securerwxl02 31 0x10000 0x100000 Non-SecurerwxlNon-secure callable region 0 placed in flash region 1 with size 32.SRAM regionDomainPermissions00 07 0x00000 0x10000 Securerwxl08 31 0x10000 0x40000 Non-SecurerwxlPeripheralDomainStatus00 NRF_P0               Non-SecureOK01 NRF_CLOCK            Non-SecureOK02 NRF_RTC0             Non-SecureOK03 NRF_RTC1             Non-SecureOK04 NRF_NVMC             Non-SecureOK05 NRF_UARTE1           Non-SecureOK06 NRF_UARTE2           SecureSKIP07 NRF_TWIM2            Non-SecureOK08 NRF_SPIM3            Non-SecureOK09 NRF_TIMER0           Non-SecureOK10 NRF_TIMER1           Non-SecureOK11 NRF_TIMER2           Non-SecureOK12 NRF_SAADC            Non-SecureOK13 NRF_PWM0             Non-SecureOK14 NRF_PWM1             Non-SecureOK15 NRF_PWM2             Non-SecureOK16 NRF_PWM3             Non-SecureOK17 NRF_WDT              Non-SecureOK18 NRF_IPC              Non-SecureOK19 NRF_VMC              Non-SecureOK20 NRF_FPU              Non-SecureOK21 NRF_EGU1             Non-SecureOK22 NRF_EGU2             Non-SecureOK23 NRF_DPPIC            Non-SecureOK24 NRF_REGULATORS       Non-SecureOK25 NRF_PDM              Non-SecureOK26 NRF_I2S              Non-SecureOK27 NRF_GPIOTE1          Non-SecureOKSPM: NS image at 0x10000SPM: NS MSP at 0x2001c908SPM: NS reset vector at 0x13299SPM: prepare to jump to Non-Secure image.*** Booting Zephyr OS build v2.6.0-rc1-ncs1  ***

/*c*/
*** Booting Zephyr OS build v2.6.0-rc1-ncs1  ***Flash regionsDomainPermissions00 01 0x00000 0x10000 Securerwxl02 31 0x10000 0x100000 Non-SecurerwxlNon-secure callable region 0 placed in flash region 1 with size 32.SRAM regionDomainPermissions00 07 0x00000 0x10000 Securerwxl08 31 0x10000 0x40000 Non-SecurerwxlPeripheralDomainStatus00 NRF_P0               Non-SecureOK01 NRF_CLOCK            Non-SecureOK02 NRF_RTC0             Non-SecureOK03 NRF_RTC1             Non-SecureOK04 NRF_NVMC             Non-SecureOK05 NRF_UARTE1           Non-SecureOK06 NRF_UARTE2           SecureSKIP07 NRF_TWIM2            Non-SecureOK08 NRF_SPIM3            Non-SecureOK09 NRF_TIMER0           Non-SecureOK10 NRF_TIMER1           Non-SecureOK11 NRF_TIMER2           Non-SecureOK12 NRF_SAADC            Non-SecureOK13 NRF_PWM0             Non-SecureOK14 NRF_PWM1             Non-SecureOK15 NRF_PWM2             Non-SecureOK16 NRF_PWM3             Non-SecureOK17 NRF_WDT              Non-SecureOK18 NRF_IPC              Non-SecureOK19 NRF_VMC              Non-SecureOK20 NRF_FPU              Non-SecureOK21 NRF_EGU1             Non-SecureOK22 NRF_EGU2             Non-SecureOK23 NRF_DPPIC            Non-SecureOK24 NRF_REGULATORS       Non-SecureOK25 NRF_PDM              Non-SecureOK26 NRF_I2S              Non-SecureOK27 NRF_GPIOTE1          Non-SecureOKSPM: NS image at 0x10000SPM: NS MSP at 0x2001c788SPM: NS reset vector at 0x12eadSPM: prepare to jump to Non-Secure image.*** Booting Zephyr OS build v2.6.0-rc1-ncs1  ***The AT host sample started





2021-10-04T20:33:35.845Z DEBUG modem >> AT+CFUN?
2021-10-04T20:33:40.936Z DEBUG modem >> AT+CFUN=1
2021-10-04T20:33:42.588Z DEBUG modem >> AT+CFUN?
2021-10-04T20:33:42.600Z DEBUG modem >> AT+CGSN=1
2021-10-04T20:33:42.617Z DEBUG modem >> AT+CGMI
2021-10-04T20:33:42.634Z DEBUG modem >> AT+CGMM
2021-10-04T20:33:42.651Z DEBUG modem >> AT+CGMR
2021-10-04T20:33:42.667Z DEBUG modem >> AT+CEMODE?
2021-10-04T20:33:42.684Z DEBUG modem >> AT%XCBAND=?
2021-10-04T20:33:42.701Z DEBUG modem >> AT+CMEE?
2021-10-04T20:33:42.718Z DEBUG modem >> AT+CMEE=1
2021-10-04T20:33:42.733Z DEBUG modem >> AT+CNEC?
2021-10-04T20:33:42.749Z DEBUG modem >> AT+CNEC=24
2021-10-04T20:33:42.757Z DEBUG modem >> AT+CGEREP?
2021-10-04T20:33:42.768Z DEBUG modem >> AT+CGDCONT?
2021-10-04T20:33:42.781Z DEBUG modem >> AT+CGACT?
2021-10-04T20:33:42.809Z DEBUG modem >> AT+CGEREP=1
2021-10-04T20:33:42.818Z DEBUG modem >> AT+CIND=1,1,1
2021-10-04T20:33:42.834Z DEBUG modem >> AT+CEREG=5
2021-10-04T20:33:42.848Z DEBUG modem >> AT+CEREG?
2021-10-04T20:33:42.869Z DEBUG modem >> AT%CESQ=1
2021-10-04T20:33:42.884Z DEBUG modem >> AT+CESQ
2021-10-04T20:33:42.901Z DEBUG modem >> AT%XSIM=1
2021-10-04T20:33:42.914Z DEBUG modem >> AT%XSIM?
2021-10-04T20:33:42.932Z DEBUG modem >> AT+CPIN?
2021-10-04T20:33:42.951Z DEBUG modem >> AT+CPINR="SIM PIN"
2021-10-04T20:33:42.969Z DEBUG modem >> AT+CIMI
2021-10-04T20:33:45.269Z DEBUG modem >> AT+CGDCONT?
2021-10-04T20:33:45.298Z DEBUG modem >> AT+CGACT?



2021-10-04T20:33:28.444Z DEBUG modem << The AT host sample started
2021-10-04T20:33:35.829Z INFO Modem port is closed
2021-10-04T20:33:35.839Z INFO Modem port is opened
2021-10-04T20:33:35.845Z DEBUG modem >> AT+CFUN?
2021-10-04T20:33:35.860Z DEBUG modem << +CFUN: 0
2021-10-04T20:33:35.861Z DEBUG modem << OK
2021-10-04T20:33:40.936Z DEBUG modem >> AT+CFUN=1
2021-10-04T20:33:40.978Z DEBUG modem << OK
2021-10-04T20:33:42.588Z DEBUG modem >> AT+CFUN?
2021-10-04T20:33:42.596Z DEBUG modem << +CFUN: 1
2021-10-04T20:33:42.599Z DEBUG modem << OK
2021-10-04T20:33:42.600Z DEBUG modem >> AT+CGSN=1
2021-10-04T20:33:42.613Z DEBUG modem << +CGSN: "352656109853344"
2021-10-04T20:33:42.614Z DEBUG modem << OK
2021-10-04T20:33:42.617Z DEBUG modem >> AT+CGMI
2021-10-04T20:33:42.630Z DEBUG modem << Nordic Semiconductor ASA
2021-10-04T20:33:42.631Z DEBUG modem << OK
2021-10-04T20:33:42.634Z DEBUG modem >> AT+CGMM
2021-10-04T20:33:42.646Z DEBUG modem << nRF9160-SICA
2021-10-04T20:33:42.648Z DEBUG modem << OK
2021-10-04T20:33:42.651Z DEBUG modem >> AT+CGMR
2021-10-04T20:33:42.663Z DEBUG modem << mfw_nrf9160_1.2.3
2021-10-04T20:33:42.664Z DEBUG modem << OK
2021-10-04T20:33:42.665Z INFO Nordic Semiconductor ASA nRF9160-SICA [mfw_nrf9160_1.2.3] SerNr: 352656109853344
2021-10-04T20:33:42.667Z DEBUG modem >> AT+CEMODE?
2021-10-04T20:33:42.680Z DEBUG modem << +CEMODE: 2
2021-10-04T20:33:42.681Z DEBUG modem << OK
2021-10-04T20:33:42.684Z DEBUG modem >> AT%XCBAND=?
2021-10-04T20:33:42.697Z DEBUG modem << %XCBAND: (1,2,3,4,5,8,12,13,18,19,20,25,26,28,66)
2021-10-04T20:33:42.698Z DEBUG modem << OK
2021-10-04T20:33:42.701Z DEBUG modem >> AT+CMEE?
2021-10-04T20:33:42.714Z DEBUG modem << +CMEE: 0
2021-10-04T20:33:42.715Z DEBUG modem << OK
2021-10-04T20:33:42.718Z DEBUG modem >> AT+CMEE=1
2021-10-04T20:33:42.730Z DEBUG modem << OK
2021-10-04T20:33:42.733Z DEBUG modem >> AT+CNEC?
2021-10-04T20:33:42.741Z DEBUG modem << +CNEC: 0
2021-10-04T20:33:42.742Z DEBUG modem << OK
2021-10-04T20:33:42.749Z DEBUG modem >> AT+CNEC=24
2021-10-04T20:33:42.755Z DEBUG modem << OK
2021-10-04T20:33:42.757Z DEBUG modem >> AT+CGEREP?
2021-10-04T20:33:42.764Z DEBUG modem << +CGEREP: 0,0
2021-10-04T20:33:42.765Z DEBUG modem << OK
2021-10-04T20:33:42.768Z DEBUG modem >> AT+CGDCONT?
2021-10-04T20:33:42.774Z DEBUG modem << OK
2021-10-04T20:33:42.781Z DEBUG modem >> AT+CGACT?
2021-10-04T20:33:42.787Z DEBUG modem << OK
2021-10-04T20:33:42.809Z DEBUG modem >> AT+CGEREP=1
2021-10-04T20:33:42.815Z DEBUG modem << OK
2021-10-04T20:33:42.818Z DEBUG modem >> AT+CIND=1,1,1
2021-10-04T20:33:42.831Z DEBUG modem << OK
2021-10-04T20:33:42.834Z DEBUG modem >> AT+CEREG=5
2021-10-04T20:33:42.840Z DEBUG modem << OK
2021-10-04T20:33:42.848Z DEBUG modem >> AT+CEREG?
2021-10-04T20:33:42.864Z DEBUG modem << +CEREG: 5,4,"FFFE","FFFFFFFF",7,0,0,"00000000","00000000"
2021-10-04T20:33:42.865Z DEBUG modem << OK
2021-10-04T20:33:42.869Z DEBUG modem >> AT%CESQ=1
2021-10-04T20:33:42.881Z DEBUG modem << OK
2021-10-04T20:33:42.884Z DEBUG modem >> AT+CESQ
2021-10-04T20:33:42.897Z DEBUG modem << +CESQ: 99,99,255,255,255,255
2021-10-04T20:33:42.898Z DEBUG modem << OK
2021-10-04T20:33:42.901Z DEBUG modem >> AT%XSIM=1
2021-10-04T20:33:42.907Z DEBUG modem << OK
2021-10-04T20:33:42.914Z DEBUG modem >> AT%XSIM?
2021-10-04T20:33:42.921Z DEBUG modem << %XSIM: 1
2021-10-04T20:33:42.922Z DEBUG modem << OK
2021-10-04T20:33:42.932Z DEBUG modem >> AT+CPIN?
2021-10-04T20:33:42.940Z DEBUG modem << +CPIN: READY
2021-10-04T20:33:42.941Z DEBUG modem << OK
2021-10-04T20:33:42.951Z DEBUG modem >> AT+CPINR="SIM PIN"
2021-10-04T20:33:42.966Z DEBUG modem << +CPINR: "SIM PIN",3
2021-10-04T20:33:42.967Z DEBUG modem << OK
2021-10-04T20:33:42.969Z DEBUG modem >> AT+CIMI
2021-10-04T20:33:42.981Z DEBUG modem << 3026xxxxxxx8958
2021-10-04T20:33:42.982Z DEBUG modem << OK
2021-10-04T20:33:42.984Z INFO IMSIdentity: 3026xxxxxxx8958
2021-10-04T20:33:43.901Z DEBUG modem << %CESQ: 42,2,3,0
2021-10-04T20:33:43.931Z DEBUG modem << +CEREG: 2,"2D83","01D11B0A",7,0,0,"11100000","11100000"
2021-10-04T20:33:45.232Z DEBUG modem << +CGEV: ME PDN ACT 0,0
2021-10-04T20:33:45.244Z DEBUG modem << +CNEC_ESM: 50,0
2021-10-04T20:33:45.269Z DEBUG modem >> AT+CGDCONT?
2021-10-04T20:33:45.272Z DEBUG modem << +CEREG: 1,"2D83","01D11B0A",7,,,"11100000","11100000"
2021-10-04T20:33:45.274Z DEBUG modem << +CIND: "service",1
2021-10-04T20:33:45.286Z DEBUG modem << +CGDCONT: 0,"IP","mnet.bell.ca.ioe","10.169.167.185",0,0
2021-10-04T20:33:45.287Z DEBUG modem << OK
2021-10-04T20:33:45.298Z DEBUG modem >> AT+CGACT?
2021-10-04T20:33:45.306Z DEBUG modem << +CGACT: 0,1
2021-10-04T20:33:45.306Z DEBUG modem << OK


IMEI: 352656109498363
PIN: 084922

SIM卡
eSIm: 893108052003577245
PUK: 14800021
PIN: 0000


*** Booting Zephyr OS build v2.6.99-ncs1  ***
Flash regions           Domain          Permissions
00 01 0x00000 0x10000   Secure          rwxl
02 31 0x10000 0x100000  Non-Secure      rwxl

Non-secure callable region 0 placed in flash region 1 with size 32.

SRAM region             Domain          Permissions
00 07 0x00000 0x10000   Secure          rwxl
08 31 0x10000 0x40000   Non-Secure      rwxl

Peripheral              Domain          Status
00 NRF_P0               Non-Secure      OK
01 NRF_CLOCK            Non-Secure      OK
02 NRF_RTC0             Non-Secure      OK
03 NRF_RTC1             Non-Secure      OK
04 NRF_NVMC             Non-Secure      OK
05 NRF_UARTE1           Non-Secure      OK
06 NRF_UARTE2           Secure          SKIP
07 NRF_TWIM2            Non-Secure      OK
08 NRF_SPIM3            Non-Secure      OK
09 NRF_TIMER0           Non-Secure      OK
10 NRF_TIMER1           Non-Secure      OK
11 NRF_TIMER2           Non-Secure      OK
12 NRF_SAADC            Non-Secure      OK
13 NRF_PWM0             Non-Secure      OK
14 NRF_PWM1             Non-Secure      OK
15 NRF_PWM2             Non-Secure      OK
16 NRF_PWM3             Non-Secure      OK
17 NRF_WDT              Non-Secure      OK
18 NRF_IPC              Non-Secure      OK
19 NRF_VMC              Non-Secure      OK
20 NRF_FPU              Non-Secure      OK
21 NRF_EGU1             Non-Secure      OK
22 NRF_EGU2             Non-Secure      OK
23 NRF_DPPIC            Non-Secure      OK
24 NRF_REGULATORS       Non-Secure      OK
25 NRF_PDM              Non-Secure      OK
26 NRF_I2S              Non-Secure      OK
27 NRF_GPIOTE1          Non-Secure      OK

SPM: NS image at 0x10000
SPM: NS MSP at 0x2002c7a0
SPM: NS reset vector at 0x26695
SPM: prepare to jump to Non-Secure image.
*** Booting Zephyr OS build v2.6.99-ncs1  ***

MOSH version:       v1.7.0
MOSH build id:      custom
MOSH build variant: dev

Initializing modemlib...


mosh:~$ Initialized modemlib


Network registration status: searching
LTE cell changed: Cell ID: 47213885, Tracking area: 7462
Currently active system mode: NB-IoT
RRC mode: Connected
mosh:~$ 
mosh:~$ 
PDN event: PDP context 0 activated
Network registration status: Connected - home network
PSM parameter update: TAU: -1, Active time: -1 seconds
mosh:~$ 
mosh:~$ 
Modem config for system mode: NB-IoT - GNSS
Modem config for LTE preference: No preference, automatically selected by the modem
Currently active system mode: NB-IoT
Modem FW version:      mfw_nrf9160_1.3.0
Operator PLMN:        "46000"
Current cell id:       47213885 (0x02D06D3D)
Current band:          8
Current rsrp:          52: -89dBm
Current snr:           29: 4dB
Mobile network time and date: 21/10/18,02:51:57+32
Could not parse dns str for cid 0, err: -22
PDP context info 1:
  CID:                    0
  PDN ID:                 0
  PDP context active:     yes
  PDP type:               IP
  APN:                    cmiot
  IPv4 MTU:               0
  IPv4 address:           10.140.99.155
  IPv6 address:           ::
  IPv4 DNS address:       0.0.0.0, 0.0.0.0
  IPv6 DNS address:       ::, ::
mosh:~$ 
RRC mode: Idle
mosh:~$ ping -d 220.181.38.149
Could not parse dns str for cid 0, err: -22
Initiating ping to: 220.181.38.149
Source IP addr: 10.140.99.155
Destination IP addr: 220.181.38.149
RRC mode: Connected
Pinging 220.181.38.149 results: time=0.567secs, payload sent: 0, payload received 0
Pinging 220.181.38.149 results: time=0.801secs, payload sent: 0, payload received 0
Pinging 220.181.38.149 results: time=0.272secs, payload sent: 0, payload received 0
Pinging 220.181.38.149 results: time=0.272secs, payload sent: 0, payload received 0

Ping statistics for 220.181.38.149:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
Approximate round trip times in milli-seconds:
    Minimum = 272ms, Maximum = 801ms, Average = 478ms
Pinging DONE
mosh:~$ ping -d 14.215.177.38
Could not parse dns str for cid 0, err: -22
Initiating ping to: 14.215.177.38
Source IP addr: 10.140.99.155
Destination IP addr: 14.215.177.38
Pinging 14.215.177.38 results: time=0.407secs, payload sent: 0, payload received 0
Pinging 14.215.177.38 results: time=0.794secs, payload sent: 0, payload received 0
Pinging 14.215.177.38 results: time=0.320secs, payload sent: 0, payload received 0
Pinging 14.215.177.38 results: time=0.217secs, payload sent: 0, payload received 0

Ping statistics for 14.215.177.38:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
Approximate round trip times in milli-seconds:
    Minimum = 217ms, Maximum = 794ms, Average = 434ms
Pinging DONE
RRC mode: Idle
mosh:~$ 









```C
	ADC_REF_MV		49152	/* 4.096V * 12x divider * 1000mV/V */

	adc_raw_to_millivolts(ADC_REF_MV, ADC_GAIN_1,16, &mv_value);
		==>	
			adc_mv = *mv_value * ADC_REF_MV;
			adc_gain_invert(gain, &adc_mv);
				==> adc_mv = adc_mv * 1/1;
		==> 
			adc_mv / 2^16
```







// trace_myself.h
#undef TRACE_SYSTEM
#define TRACE_SYSTEM myself
#if !defined(__TRACE_MYSELF_H__) || defined(TRACE_HEADER_MULTI_READ)#define __TRACE_MYSELF_H__#include <linux/tracepoint.h> // 此处是非常关键的地方，设计到你要追踪的函数的的相关内容作为参数// 为了方便，这里将参数设置为unsiged short形式TRACE_EVENT(myself_tp,
    TP_PROTO(unsigned short dest, unsigned short source),
    TP_ARGS(dest, source), // 定义两参数名称为dest和source
    TP_STRUCT__entry(  // 此处本人理解为打桩时候分配的环形队列时指定的作用域，说白了就是大小和attr。
        __field(unsigned short, dest)
        __field(unsigned short, source)
    ),
TP_fast_assign(
    __entry->dest = dest;   //  将trace的函数的内容拷贝到环形队列中去
    __entry->source = source;
),

TP_printk("dest:%d, source:%d", __entry->dest, __entry->source)  // 打印你所期望的内容
);

#endif // 此处定义完成后，仅仅是类型定义成功
// 下一步我们需要指定头文件所在的目录，并且定义头文件的名称#undef TRACE_INCLUDE_PATH
#define TRACE_INCLUDE_PATH .
#define TRACE_INCLUDE_FILE trace_myself // 这就是该头文件的名字
#include <trace/define_trace.h>




BK9531:
bk9531.c         319 [D] bk9531 reg = 0x38 , reg_value = 0x00000000
bk9531.c         319 [D] bk9531 reg = 0x39 , reg_value = 0x03d7d5f7


bk9531.c         319 [D] bk9531 reg = 0x38 , reg_value = 0x00000000
bk9531.c         319 [D] bk9531 reg = 0x39 , reg_value = 0x03d7d5f7


BK9532

bk9532.c         879 [D] bk9532 reg = 0x38 , reg_value = 0x40d7d5f7
bk9532.c         879 [D] bk9532 reg = 0x39 , reg_value = 0x00000000

bk9532.c         879 [D] bk9532 reg = 0x38 , reg_value = 0x40d7d5f7
bk9532.c         879 [D] bk9532 reg = 0x39 , reg_value = 0x00000000

bk9532.c         879 [D] bk9532 reg = 0x38 , reg_value = 0x4fd7d5f7
bk9532.c         879 [D] bk9532 reg = 0x39 , reg_value = 0x00000000


bk9532.c         879 [D] bk9532 reg = 0x32 , reg_value = 0x20ff0f09


/usr/bin/qemu-arm-static: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, BuildID[sha1]=dbabf65e9839ab06800c13572926717ac0249c75, for GNU/Linux 3.2.0, stripped

/usr/bin/qemu-arm-static: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, for GNU/Linux 3.2.0, BuildID[sha1]=924679b2e711c1e6baf83e052ee913894bd03e33, stripped







altbootcmd=run bootcmd
arch=arm
autoload=no
baudrate=115200
board=stm32mp1
board_name=stm32mp135f-dk
boot_a_script=load ${devtype} ${devnum}:${distro_bootpart} ${scriptaddr} ${prefix}${script}; source ${scriptaddr}
boot_device=mmc
boot_efi_binary=if fdt addr ${fdt_addr_r}; then bootefi bootmgr ${fdt_addr_r};else bootefi bootmgr ${fdtcontroladdr};fi;load ${devtype} ${devnum}:${distro_bootpart} ${kernel_addr_r} efi/boot/bootarm.efi; if fdt addr ${fdt_addr_r}; then bootefi ${kernel_addr_r} ${fdt_addr_r};else bootefi ${kernel_addr_r} ${fdtcontroladdr};fi
boot_extlinux=sysboot ${devtype} ${devnum}:${distro_bootpart} any ${scriptaddr} ${prefix}${boot_syslinux_conf}
boot_instance=0
boot_net_usb_start=true
boot_prefixes=/ /boot/
boot_script_dhcp=boot.scr.uimg
boot_scripts=boot.scr.uimg boot.scr
boot_syslinux_conf=extlinux/extlinux.conf
boot_targets=mmc0
bootcmd=run bootcmd_stm32mp
bootcmd_mmc0=devnum=0; run mmc_boot
bootcmd_mmc1=devnum=1; run mmc_boot
bootcmd_mmc2=devnum=2; run mmc_boot
bootcmd_pxe=run boot_net_usb_start; dhcp; if pxe get; then pxe boot; fi
bootcmd_stm32mp=echo "Boot over ${boot_device}${boot_instance}!";if test ${boot_device} = serial || test ${boot_device} = usb;then stm32prog ${boot_device} ${boot_instance}; else run env_check;if test ${boot_device} = mmc;then env set boot_targets "mmc${boot_instance}"; fi;if test ${boot_device} = nand || test ${boot_device} = spi-nand ;then env set boot_targets ubifs0; fi;if test ${boot_device} = nor;then env set boot_targets mmc0; fi;run distro_bootcmd;fi;
bootcmd_ubifs0=devnum=0; run ubifs_boot
bootcount=3
bootdelay=3
cpu=armv7
distro_bootcmd=for target in ${boot_targets}; do run bootcmd_${target}; done
efi_dtb_prefixes=/ /dtb/ /dtb/current/
env_check=if env info -p -d -q; then env save; fi
fdt_addr_r=0xc4000000
fdtcontroladdr=dabf1ea0
fdtfile=stm32mp135f-dk.dtb
fdtoverlay_addr_r=0xc4100000
kernel_addr_r=0xc2000000
load_efi_dtb=load ${devtype} ${devnum}:${distro_bootpart} ${fdt_addr_r} ${prefix}${efi_fdtfile}
loadaddr=0xc2000000
mmc_boot=if mmc dev ${devnum}; then devtype=mmc; run scan_dev_for_boot_part; fi
pxefile_addr_r=0xc4200000
ramdisk_addr_r=0xc4400000
scan_dev_for_boot=echo Scanning ${devtype} ${devnum}:${distro_bootpart}...; for prefix in ${boot_prefixes}; do run scan_dev_for_extlinux; run scan_dev_for_scripts; done;run scan_dev_for_efi;
scan_dev_for_boot_part=part list ${devtype} ${devnum} -bootable devplist; env exists devplist || setenv devplist 1; for distro_bootpart in ${devplist}; do if fstype ${devtype} ${devnum}:${distro_bootpart} bootfstype; then run scan_dev_for_boot; fi; done; setenv devplist
scan_dev_for_efi=setenv efi_fdtfile ${fdtfile}; if test -z "${fdtfile}" -a -n "${soc}"; then setenv efi_fdtfile ${soc}-${board}${boardver}.dtb; fi; for prefix in ${efi_dtb_prefixes}; do if test -e ${devtype} ${devnum}:${distro_bootpart} ${prefix}${efi_fdtfile}; then run load_efi_dtb; fi;done;if test -e ${devtype} ${devnum}:${distro_bootpart} efi/boot/bootarm.efi; then echo Found EFI removable media binary efi/boot/bootarm.efi; run boot_efi_binary; echo EFI LOAD FAILED: continuing...; fi; setenv efi_fdtfile
scan_dev_for_extlinux=if test -e ${devtype} ${devnum}:${distro_bootpart} ${prefix}${boot_syslinux_conf}; then echo Found ${prefix}${boot_syslinux_conf}; run boot_extlinux; echo SCRIPT FAILED: continuing...; fi
scan_dev_for_scripts=for script in ${boot_scripts}; do if test -e ${devtype} ${devnum}:${distro_bootpart} ${prefix}${script}; then echo Found U-Boot script ${prefix}${script}; run boot_a_script; echo SCRIPT FAILED: continuing...; fi; done
scriptaddr=0xc4100000
serial#=800F801C3530510438343532
serverip=192.168.1.1
soc=stm32mp
splashimage=0xc4300000
ubifs_boot=env exists bootubipart || env set bootubipart UBI; env exists bootubivol || env set bootubivol boot; if ubi part ${bootubipart} && ubifsmount ubi${devnum}:${bootubivol}; then devtype=ubi; run scan_dev_for_boot; fi
usb_boot=usb start; if usb dev ${devnum}; then devtype=usb; run scan_dev_for_boot_part; fi
vendor=st





gst-launch-1.0 videotestsrc num-buffers=250 \
 ! 'video/x-raw,format=(string)I420,width=320,height=240,framerate=(fraction)25/1' \
 ! queue ! mux. \
 audiotestsrc num-buffers=440 ! audioconvert \
 ! 'audio/x-raw,rate=44100,channels=2' ! queue ! mux. \
 avimux name=mux ! filesink location=test.avi

gst-launch-1.0 v4l2src ! "video/x-raw, format=RGB16, width=640, height=480, framerate=(fraction)30/1" ! queue ! waylandsink fullscreen=true -e

gst-launch-1.0 v4l2src ! "video/x-raw, format=RGB16, width=640, height=480, framerate=(fraction)30/1" ! queue ! videorate ! filesink location=test.avi -e	//可以运行

gst-launch-1.0 videotestsrc ! "video/x-raw, format=RGB16, width=640, height=480, framerate=(fraction)30/1" ! queue ! videoconvert ! filesink location=test.avi -e 	//可以运行

gst-launch-1.0 -e avimux name=mux1 ! filesink location=test.avi v4l2src device=/dev/video0 ! video/x-h264, framerate=30/1, width=640, height=360 ! queue ! mux1. alsasrc device=hw:1,0 ! audio/x-raw, rate=32000, channels=2, layout=interleaved, format=S16LE ! queue ! mux1.

gst-launch-1.0 -e avimux name=mux1 ! filesink location=test.avi v4l2src device=/dev/video0 ! video/x-raw, format=RGB16, framerate=30/1, width=640, height=480 ! queue ! mux1. alsasrc device=hw:0,0 ! audio/x-raw, rate=48000, channels=2, layout=interleaved, format=S32LE ! queue ! mux1.


gst-launch-1.0 videotestsrc ! autovideosink

gst-launch-1.0 v4l2src device=/dev/video0  ! video/x-raw,format=RGB16,width=640,height=480,framerate=20/1  !  autovideosink	//可行
gst-launch-1.0 v4l2src device=/dev/video0  ! video/x-raw,format=RGB16,width=640,height=480,framerate=20/1 ! filesink location=test.avi	//可行

gst-launch-1.0 v4l2src device=/dev/video0  ! video/x-raw,width=640,height=480,framerate=20/1  ! avimux !  filesink location=test.avi	//ubuntu上可行

gst-launch-1.0 v4l2src device=/dev/video0  ! video/x-raw,format=RGB16,width=640,height=480,framerate=20/1  ! videoconvert ! avimux !  filesink location=test.avi -e	//st135可行

gst-play-1.0 playbin test.avi	//播放




gst-launch-1.0 v4l2src device=/dev/video0  ! video/x-raw,width=640,height=480,framerate=20/1  ! avimux !  filesink location=test.avi

gst-launch-1.0 v4l2src device=/dev/video0 ! 'video/x-raw,format=RGB16,width=640,height=480,framerate=20/1' ! ffmpegcolorspace ! ximagesink

gst-launch-0.10 v4l2src !  video/x-raw-yuv,width=352,height=288 ! xvimagesink


gst-launch-1.0 -e v4l2src ! "video/x-raw, format=RGB16, width=640, height=480, framerate=(fraction)30/1" ! teename=srctee srctee. ! queue2 name=squeue ! ffmpegcolorspace ! xvimagesink srctee. ! queue2 name=fqueue ! videorate ! ffmpegcolorspace ! ffenc_mpeg4 ! avimux ! filesink location=test.avi






lr1110_bootloader_write_flash_encrypted( context, local_offset, buffer + loop * 64, min( remaining_length, 64 ) );
	-> uint8_t cdata[256];
	-> cbuffer[6];
	-> lr1110_bootloader_fill_cbuffer_cdata_flash( cbuffer, cdata, 0x8003, offset, buffer + loop * 64, 64 );
		-> lr1110_bootloader_fill_cbuffer_opcode_offset_flash( cbuffer, 0x8003, offset ); => cbuffer = 0x8003 + offset
		-> lr1110_bootloader_fill_cdata_flash( cdata, buffer + loop * 64, 64 );	=> uint32_t 填充到 uint8_t


openocd -f D:/WorkSoftware/STM32_ENV/openocd-0.10.0/scripts/interface/stlink-v2.cfg -f D:/WorkSoftware/STM32_ENV/openocd-0.10.0/scripts/target/stm32f0x.cfg -c init -c targets -c "reset halt" -c "flash write_image erase ./app.hex" -c "reset halt" -c "verify_image ./app.hex" -c "reset run" -c shutdown

openocd -f interface/jlink.cfg -f target/stm32f0x.cfg  -c init -c targets -c "reset halt" -c "flash write_image erase FT32F0XX.hex" -c "reset halt" -c "verify_image FT32F0XX.hex" -c "reset run" -c shutdown

openocd -f stm32f030_jlink.cfg -c init -c targets -c 'reset halt' -c 'flash write_image erase FT32F0XX.hex' -c 'reset halt' -c 'verify_image FT32F0XX.hex' -c 'reset run' -c shutdown


this->demo_wifi_settings_default.channels              = DEMO_WIFI_CHANNELS_DEFAULT >> 1;
this->demo_wifi_settings_default.types                 = DEMO_WIFI_TYPE_SCAN_DEFAULT;
this->demo_wifi_settings_default.scan_mode             = DEMO_WIFI_MODE_DEFAULT;
this->demo_wifi_settings_default.nbr_retrials          = DEMO_WIFI_NBR_RETRIALS_DEFAULT;	// 5
this->demo_wifi_settings_default.max_results           = DEMO_WIFI_MAX_RESULTS_DEFAULT;		// 15
this->demo_wifi_settings_default.timeout               = DEMO_WIFI_TIMEOUT_IN_MS_DEFAULT;	// 110
this->demo_wifi_settings_default.result_type           = DEMO_WIFI_RESULT_TYPE_DEFAULT;
this->demo_wifi_settings_default.does_abort_on_timeout = DEMO_WIFI_DOES_ABORT_ON_TIMEOUT_DEFAULT;


