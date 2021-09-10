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


mipi_dsi_host_register 
mipi_dsi_host *host->dev->of_node->full_name = dsi@5a000000
OF: pp->name=#address-cells
OF: pp->name=#size-cells
OF: pp->name=name
OF: pp->name=#address-cells
OF: pp->name=#size-cells
OF: pp->name=name
skip nodes without reg property
for_each_available_child_of_node done
OF: pp->name=reg
OF: pp->name=remote-endpoint
OF: pp->name=phandle
OF: pp->name=name
OF: pp->name=reg
OF: pp->name=remote-endpoint
OF: pp->name=phandle
OF: pp->name=name
OF: pp->name=remote-endpoint
OF: pp->name=compatible
OF: pp->name=reg
OF: pp->name=reset-gpios
OF: pp->name=power-supply
OF: pp->name=status
OF: pp->name=compatible
OF: pp->name=reg
OF: pp->name=reset-gpios
OF: pp->name=power-supply
OF: pp->name=status
OF: pp->name=compatible
OF: pp->name=compatible
OF: pp->name=compatible
OF: pp->name=compatible

<stdout>: Warning (avoid_unnecessary_addr_size): /soc/dsi@5a000000: unnecessary #address-cells/#size-cells without "ranges" or child "reg" property

for_each_available_child_of_node done
OF: graph: no port node found in /soc/dsi@5a000000/ports



struct mipi_dsi_host *host;
struct mipi_dsi_host *host


static int rpi_touchscreen_probe(struct i2c_client *i2c,
				 const struct i2c_device_id *id)
{
	struct device *dev = &i2c->dev;
	struct rpi_touchscreen *ts;
	struct device_node *endpoint, *dsi_host_node;
	struct mipi_dsi_host *host;
	int ver;
	struct mipi_dsi_device_info info = {
		.type = RPI_DSI_DRIVER_NAME,
		.channel = 0,
		.node = NULL,
	};
	printk("rpi_touchscreen_probe be called...\n");

	ts = devm_kzalloc(dev, sizeof(*ts), GFP_KERNEL);
	if (!ts)
	{
		return -ENOMEM;
	}

	i2c_set_clientdata(i2c, ts);

	ts->i2c = i2c;

	ver = rpi_touchscreen_i2c_read(ts, REG_ID);
	if (ver < 0)
		ver = rpi_touchscreen_i2c_read(ts, REG_ID);
	if (ver < 0) {
		dev_err(dev, "Atmel I2C read failed: %d\n", ver);
		return -ENODEV;
	}

	switch (ver) {
	case 0xde: /* ver 1 */
	case 0xc3: /* ver 2 */
		break;
	default:
		dev_err(dev, "Unknown Atmel firmware revision: 0x%02x\n", ver);
		return -ENODEV;
	}

	/* Turn off at boot, so we can cleanly sequence powering on. */
	rpi_touchscreen_i2c_write(ts, REG_POWERON, 1);

	/* Turn on the backlight. */
	rpi_touchscreen_i2c_write(ts, REG_PWM, 255);

	printk("open REG_PWM\n");

	/* Look up the DSI host.  It needs to probe before we do. */
	endpoint = of_graph_get_next_endpoint(dev->of_node, NULL);
	if (!endpoint)
	{
		return -ENODEV;
	}
		
	printk("endpoint = %pOF\n",endpoint);

	dsi_host_node = of_graph_get_remote_port_parent(endpoint);
	if (!dsi_host_node)
	{
		goto error;
	}

	printk("dsi_host_node = %pOF\n",dsi_host_node);

#if 1
	host = of_find_mipi_dsi_host_by_node(dsi_host_node);
	of_node_put(dsi_host_node);
	if (!host) {
		of_node_put(endpoint);
		printk("of_find_mipi_dsi_host_by_node fail\n");
		return -EPROBE_DEFER;
	}
#endif


#if 0
	info.node = of_graph_get_remote_port(endpoint);
	if (!info.node)
	{
		printk("of_graph_get_remote_port fail\n");
		goto error;
	}

	of_node_put(endpoint);


	ts->dsi = mipi_dsi_device_register_full(host, &info);
	if (IS_ERR(ts->dsi)) {
		dev_err(dev, "DSI device registration failed: %ld\n",
			PTR_ERR(ts->dsi));
		return PTR_ERR(ts->dsi);
	}
#endif

	drm_panel_init(&ts->base, dev, &rpi_touchscreen_funcs,
		       DRM_MODE_CONNECTOR_DSI);

	/* This appears last, as it's what will unblock the DSI host
	 * driver's component bind function.
	 */
	drm_panel_add(&ts->base);

	printk("pi_touchscreen_probe success\n");
	return 0;

error:
	of_node_put(endpoint);
	return -ENODEV;
}




panel.c  
static int panel_bridge_attach(struct drm_bridge *bridge,
			       enum drm_bridge_attach_flags flags)
|
drm_bridge.c
drm_bridge_attach(struct drm_encoder *encoder, struct drm_bridge *bridge,
		      struct drm_bridge *previous,
		      enum drm_bridge_attach_flags flags)
|
dw-mipi-dsi.c
static int dw_mipi_dsi_bridge_attach(struct drm_bridge *bridge,
				     enum drm_bridge_attach_flags flags)


[09-19-25.698][  OK  ] Mounted Kernel Configuration File System.
[09-19-25.716][    7.835632] systemd[1]: Started Apply Kernel Variables.
[09-19-25.732][  OK  ] Started Apply Kernel Variables.
[09-19-25.820][    7.934602] systemd[1]: Started File System Check on Root Device.
[09-19-25.831][  OK  ] Started File System Check on Root Device.
[09-19-25.848][    7.958461] systemd[1]: Starting Remount Root and Kernel File Systems...
[09-19-25.865][    7.980311] EXT4-fs (mmcblk0p4): recovery complete
[09-19-25.869][    7.984052] EXT4-fs (mmcblk0p4): mounted filesystem with ordered data mode. Opts: (null)
[09-19-25.881]         Starting Remount Root and Kernel File Systems...
[09-19-25.899][    8.011153] ext4 filesystem being mounted at /boot supports timestamps until 2038 (0x7fffffff)
[09-19-25.954][    8.069451] systemd[1]: Started Journal Service.
[09-19-25.987][  OK  ] Started Journal Service.
[09-19-26.026][    8.140935] EXT4-fs (mmcblk0p5): re-mounted. Opts: (null)
[09-19-26.133][  OK  ] Started Remount Root and Kernel File Systems.
[09-19-26.172][  OK  ] Started Starts Psplash Boot screen.
[09-19-26.231][    8.345794] [rpi_touchscreen_get_modes] mode num = 4
[09-19-26.264][    8.375475] [dw_mipi_dsi_bridge_mode_set] bridge node is /soc/dsi@5a000000 
[09-19-26.264][    8.375496] dw_mipi_dsi_mode_set be called
[09-19-26.271]         Starting Flush Journal to Persistent Storage...
[09-19-26.281][    8.393829] mode_flags = 0, lanes = 0, format = 0, lane_mbps = 0
[09-19-26.282][    8.398568] dw_mipi_dsi_get_lane_mbps be call...
[09-19-26.315]         Starting Create Static Device Nodes in /dev...
[09-19-26.331][    8.446590] [dw_mipi_dsi_get_lane_mbps] mode->clock = 25826,bpp = 24
[09-19-26.363][    8.476894] [drm] pll_out_khz = 0
[09-19-26.363][    8.478764] [drm] Warning min phy mbps is used
[09-19-26.385][    8.501873] systemd-journald[142]: Received client request to flush runtime journal.
[09-19-26.398][    8.502872] pll_in 24000kHz pll_out 63000kHz lane_mbps 63MHz
[09-19-26.404][  OK  ] Started Flush Journal to Persistent Storage.
[09-19-26.446][    8.560920] [dw_mipi_dsi_get_lane_mbps] success...
[09-19-26.449][    8.567742] [dw_mipi_dsi_phy_init] success
[09-19-26.526][    8.640968] [dw_mipi_dsi_mode_set] finish
[09-19-26.555][  OK  ] Started Create Static Device Nodes in /dev.0379] 8<--- cut here ---
[09-19-26.555]
[09-19-26.564][    8.676929] Unable to handle kernel NULL pointer dereference at virtual address 000001e4
[09-19-26.587][  OK  ] Reached target Local File Systems (Pre).
[09-19-26.598][    8.710894] pgd = 900d97e6
[09-19-26.598][    8.712166] [000001e4] *pgd=c3453835, *pte=00000000, *ppte=00000000
[09-19-26.602][    8.718469] Internal error: Oops: 17 [#1] PREEMPT SMP ARM
[09-19-26.604][    8.723867] Modules linked in:
[09-19-26.614][    8.726931] CPU: 0 PID: 188 Comm: psplash-drm Not tainted 5.10.10-gfde92c7681c1-dirty #87
[09-19-26.618][    8.735118] Hardware name: STM32 (Device Tree Support)
[09-19-26.631][    8.740260] PC is at mipi_dsi_generic_write+0x8/0xa4
[09-19-26.631][    8.745223] LR is at rpi_touchscreen_enable+0xa4/0x2ac
[09-19-26.635][    8.750364] pc : [<c0709a30>]    lr : [<c0711b00>]    psr: 200e0113
[09-19-26.647][    8.756645] sp : c3421d04  ip : c1ab0d80  fp : 00000050
[09-19-26.648][    8.761880] r10: 00000010  r9 : 00000002  r8 : 00000064
[09-19-26.652][    8.767116] r7 : 00000005  r6 : 00000001  r5 : c1c3d1c0  r4 : 00000000
[09-19-26.664][    8.773660] r3 : 00000003  r2 : 00000006  r1 : c3421d0e  r0 : 00000000
[09-19-26.666][    8.780207] Flags: nzCv  IRQs on  FIQs on  Mode SVC_32  ISA ARM  Segment none
[09-19-26.671][    8.787360] Control: 10c5387d  Table: c326406a  DAC: 00000051
[09-19-26.681][    8.793125] Process psplash-drm (pid: 188, stack limit = 0xe5904c80)
[09-19-26.682][    8.799490] Stack: (0xc3421d04 to 0xc3422000)
[09-19-26.697][    8.803859] 1d00:          c0711b00 0331ad4f 02100002 00000003 c1104d08 c2b1e840 c1c3d1c0
[09-19-26.699][    8.812062] 1d20: c307ac40 c2a761c0 c2a761c0 c0e4fa8c c1a38840 c0ca4a3c 00000001 c0707280
[09-19-26.705][    8.820265] 1d40: c1e1d840 c307ac40 c2a761c0 c06e9ae4 00000000 c307ac40 00000000 c06cd6f4


[09-44-48.853][   11.619034] [dw_mipi_dsi_phy_init] success
[09-44-48.856][  OK  ] Started Flush Journal to Persistent Storage.
[09-44-48.904][  OK  ] Started Create Static Device Nodes in /dev.
[09-44-48.923][  OK  ] Reached target Local File Systems (Pre).
[09-44-48.923][   11.690402] [dw_mipi_dsi_mode_set] finish
[09-44-48.923][   11.693037] 8<--- cut here ---
[09-44-48.937][   11.696022] Unable to handle kernel NULL pointer dereference at virtual address 0000019c
[09-44-48.954]         Mounting /var/volatile...
[09-44-48.954][   11.723973] pgd = 66748a1c
[09-44-48.956][   11.726661] [0000019c] *pgd=c328a835, *pte=00000000, *ppte=00000000
[09-44-48.988][   11.759133] Internal error: Oops: 17 [#1] PREEMPT SMP ARM
[09-44-48.990][   11.763115] Modules linked in:
[09-44-49.004][   11.766178] CPU: 1 PID: 228 Comm: psplash-drm Not tainted 5.10.10-gfde92c7681c1-dirty #88
[09-44-49.004][   11.774343] Hardware name: STM32 (Device Tree Support)
[09-44-49.022][   11.779507] PC is at rpi_touchscreen_enable+0x40/0x2c8
[09-44-49.022][   11.784666] LR is at drm_panel_enable+0x28/0xd4
[09-44-49.023][   11.789196] pc : [<c0b86c1c>]    lr : [<c0707280>]    psr: a00f0113
[09-44-49.024][   11.795454] sp : c32c1d08  ip : c0eab354  fp : 00000001
[09-44-49.036][   11.800684] r10: c0ca4a3c  r9 : c1a4a840  r8 : c0e4fa8c
[09-44-49.037][   11.805918] r7 : c2a741c0  r6 : c2a741c0  r5 : c283c240  r4 : c283c240
[09-44-49.053][   11.812461] r3 : 00000000  r2 : 00000000  r1 : c2824840  r0 : c0e59fe4
[09-44-49.053][   11.819030] Flags: NzCv  IRQs on  FIQs on  Mode SVC_32  ISA ARM  Segment none
[09-44-49.055][   11.826156] Control: 10c5387d  Table: c4bf806a  DAC: 00000051
[09-44-49.070][   11.831922] Process psplash-drm (pid: 228, stack limit = 0x4e4735e0)
[09-44-49.070][   11.838305] Stack: (0xc32c1d08 to 0xc32c2000)
[09-44-49.088][   11.842662] 1d00:                   b8fc5f90 00000002 00000000 c1104d08 c2a86440 c283c240
[09-44-49.089][   11.850873] 1d20: c32fbcc0 c2a741c0 c2a741c0 c0e4fa8c c1a4a840 c0ca4a3c 00000001 c0707280
[09-44-49.103][   11.859058] 1d40: c2824840 c32fbcc0 c2a741c0 c06e9ae4 00000000 c32fbcc0 00000000 c06cd6f4
[09-44-49.103][   11.867237] 1d60: c32fbcc0 c2940400 00000002 00000000 00000000 c3353700 c4557980 c06ce8a4
[09-44-49.120][   11.875452] 1d80: c32fbcc0 ae59b76d 00000002 c06cef40 c32fbcc0 00000000 c2940400 00000000
[09-44-49.120][   11.883649] 1da0: c335a480 c06cfc4c c32fbcc0 00000000 c32c1e1c c32fb040 c335a480 c06cedf0
[09-44-49.122][   11.891861] 1dc0: 00000000 c32c1edc c2940400 c06dbf00 c335a480 c10677d5 c10677a8 c32c1edc
[09-44-49.137][   11.900033] 1de0: c1a3a040 c2a86440 c2a86474 00000020 c32c0000 c32fb040 c4557980 00000000
[09-44-49.138][   11.908233] 1e00: c335a480 c2a86440 c4557980 00000000 00000000 c32fb040 00000001 c3236540
[09-44-49.154][   11.916464] 1e20: 00000002 00000005 00000000 00000000 c2824860 c2940518 00000100 c2940400
[09-44-49.155][   11.924632] 1e40: cccccccc c2940524 00000023 00000000 c2a86474 c06e6fbc c32c1edc c1104d08
[09-44-49.172][   11.932830] 1e60: c3353700 00000000 c2940400 00000002 c06dbd1c c32c1edc c3353700 bef78bf8
[09-44-49.172][   11.941029] 1e80: 00000068 c06d7120 00000000 c1104d08 c06864a2 00000068 c0c9a15c c06864a2
[09-44-49.187][   11.949228] 1ea0: c32c1edc 000000a2 c3353700 c06d7354 0000e200 00000001 c0e51fd0 c010fc40
[09-44-49.189][   11.957426] 1ec0: 00000001 c32c1edc 00000068 c40c3840 c06dbd1c 00000051 00000000 0003b49c
[09-44-49.203][   11.965623] 1ee0: 00000000 00000001 00000023 00000026 00000000 00000000 00000000 00000001
[09-44-49.205][   11.973821] 1f00: 00006506 03210320 03500323 01e00000 01e901e7 000001fd 0000003c 00000000
[09-44-49.221][   11.982019] 1f20: 00000048 78303038 00303834 00000000 00000000 00000000 00000000 00000000
[09-44-49.222][   11.990218] 1f40: 00000000 00000800 00000255 c0114a58 00000000 c32c1f54 c32c1f54 c1104d08
[09-44-49.239][   11.998417] 1f60: c1894200 fffffdfd c40c3840 c06864a2 bef78bf8 c40c3840 fffffdfd 00000036
[09-44-49.239][   12.006616] 1f80: 00000000 c03031b0 00000004 bef78bf8 c06864a2 00000036 c0100264 c32c0000
[09-44-49.256][   12.014814] 1fa0: 00000036 c0100060 00000004 bef78bf8 00000004 c06864a2 bef78bf8 00000001
[09-44-49.256][   12.023012] 1fc0: 00000004 bef78bf8 c06864a2 00000036 00000026 00000023 00028efa 00000000
[09-44-49.268][   12.031210] 1fe0: 4168a094 bef78bdc 416745ef 410c6118 000e0030 00000004 00000000 00000000
[09-44-49.272][   12.039434] [<c0b86c1c>] (rpi_touchscreen_enable) from [<c0707280>] (drm_panel_enable+0x28/0xd4)

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

