# Lichee pi zero firmware
1) Install Ubuntu18.04LTS (long support version)
2) install arm Linux hardware floating-point tool chain, because V3s supports VFPv4 floating-point units: 

sudo apt install gcc-arm-linux-gnueabihf

3) Install git source management software: 

sudo apt install git
    
4) Other supporting software

sudo apt install device-tree-compiler build-essential libncurses5-dev libncursesw5-dev libusb-1.0-0-dev python flex bison libssl-dev

5) Get the u-boot source code: 

For SPI Flash version

git clone -b v3s-spi-experimental https://github.com/Lichee-Pi/u-boot.git

for TF

git clone https://github.com/Lichee-Pi/u-boot.git -b v3s-current

6) Modify include/configs/sun8i.h so that u-boot can boot directly from the tf card:

nano include/configs/sun8i.h

add 

For TF card boot

#define CONFIG_BOOTCOMMAND   "setenv bootm_boot_mode sec; " \
                            "load mmc 0:1 0x41000000 zImage; "  \
                            "load mmc 0:1 0x41800000 sun8i-v3s-licheepi-zero-dock.dtb; " \
                            "bootz 0x41000000 - 0x41800000;"

#define CONFIG_BOOTARGS      "console=ttyS0,115200 panic=5 rootwait root=/dev/mmcblk0p2 earlyprintk rw  vt.global_cursor_default=0"


or for SPI Flash boot

#define CONFIG_BOOTCOMMAND   "sf probe 0; "                           \
                            "sf read 0x41800000 0x100000 0x10000; "  \
                            "sf read 0x41000000 0x110000 0x400000; " \
                            "bootz 0x41000000 - 0x41800000"
#define CONFIG_BOOTARGS      "console=ttyS0,115200 earlyprintk panic=5 rootwait " \
                            "mtdparts=spi32766.0:1M(uboot)ro,64k(dtb)ro,4M(kernel)ro,-(rootfs) root=31:03 rw rootfstype=jffs2"


before 
#endif /* __CONFIG_H */



7) Configure

cd u-boot

make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- LicheePi_Zero_800x480LCD_defconfig

make ARCH=arm menuconfig

8) Configure menu config

Architecture select - ARM

ARM architecture

Target select (Support sunxi (Allwinner) SoCs) 
(360) sunxi dram clock speed          config dram speed
(14779) sunxi dram zq value           config dram ZQ valu
-*- Board uses DDR2 DRAM             use DDR2 DRAM使用DDR2 DRAM
[*] Enable graphical uboot console on HDMI, LCD or VGA
[ ] VGA via LCD controller support
(x:800,y:480,depth:18,pclk_khz:33000,le:87,ri:40,up:31,lo:13,hs:1,vs:1,sync:3,vmode:0) LCD pane
(1)   LCD panel display clock phase
()    LCD panel power enable pin               
()    LCD panel reset pin                                            
(PB4) LCD panel backlight pwm pin                   
[*]   LCD panel backlight pwm is inverted           
[ ]   LCD panel needs to be configured via i2c
   LCD panel support (Generic parallel interface LCD panel)  --->
           (X) Generic parallel interface LCD panel                  
           ( ) Generic lvds interface LCD panel        
           ( ) MIPI 4-lane, 513Mbps LCD panel via SSD2828 bridge chip
           ( ) eDP 4-lane, 1.62G LCD panel via ANX9804 bridge chip
           ( ) Hitachi tx18d42vm LCD panel
           ( ) tl059wv5c0 LCD panel
(0) GMAC Transmit Clock Delay Chain

Boot images - (1008000000) CPU clock frequency
delay in seconds before automatically booting - 2s

SPL / TPL --->

[*]   MMC raw mode: by sector                      
(0x50)  Address on the MMC to load U-Boot from
[*] Support GPIO                                
[ ] Support I2C                                
[*] Support common libraries                   
[*] Support disk paritions                     
[*] Support generic libraries                  
[*] Support MMC                                 
[*] Support power drivers                  
[*] Support serial                               
[ ] Support USB host drivers
[ ] Support USB Gadget drivers
[ ] Support USB Ethernet drivers

9) Compile u-boot

make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j3


10) Compile kernel

cd ..    
git clone https://github.com/Lichee-Pi/linux.git -b zero-5.2.y
cd linux

make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- licheepi_zero_defconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j4
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- dtbs

11) Compile root file system

cd ..
wget https://buildroot.org/downloads/buildroot-2020.02.tar.gz
tar xvzf buildroot-2020.02.tar.gz
cd buildroot-2020.02

make menuconfig

Target options->
Target Architecture -> ARM (little endian)


Other Images: 
https://www.licheepizero.us/downloads/lichee_zero_test_Debian_LXDE.tar.bz2 - boots with RGB LCD enabled -root password is unknown

From
https://github.com/petit-miner/Blueberry-PI/wiki 
https://drive.google.com/open?id=1GIm1OjY2lM_PxxdrHHVd2gWKwr01NvFa - with root/root credentians pair - boot to console only without LCD enabled
