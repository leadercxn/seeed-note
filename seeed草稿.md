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

