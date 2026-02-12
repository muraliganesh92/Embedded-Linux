Basic Commands.



**remove directory : rm -rf directory\_name**

**create directory : mkdir directoryname**



Embedded Linux Set up



**//Upgrade Host Linux**

1\. sudo apt update

   sudo apt upgrade -y



1.1 Install Cross Compilers:



**arm-linux-gnueabihf-gcc**



1.2 Install Cross Toolchain



**sudo apt install gcc-arm-linux-gnueabihf g++-arm-linux-gnueabihf		//older tool chain**

**sudo apt install gcc-8-arm-linux-gnueabihf g++-8-arm-linux-gnueabihf		//Latest one**



**sudo apt update \&\& sudo apt install -y build-essential gettext libncurses-dev python3 rsync wget unzip bc**



**------------------------------------------------------------------------------------------------------------------------------------------**

**///////////////////////////////////////////////////////Building from predefined files/////////////////////////////////////////////////////**

**------------------------------------------------------------------------------------------------------------------------------------------**





1.3 Incase for booting from predefined images,



A -> go to, https://beagleboard.org

B -> Download the images in the above site.

C -> Install the xz utilities.



**sudo apt install xz-utils**



D -> write the below command to get the .img files from .xz file



**xz -d <image>.img.xz**



for example, In the beaglebone site, we get the files as,



muraliganesh92@GaneshLinux:~/Downloads$ ls

**am335x-debian-13.2-base-v6.12-armhf-2025-12-08-4gb.img.xz**



and after using the command,



**xz -d am335x-debian-13.2-base-v6.12-armhf-2025-12-08-4gb.img.xz**



we get the output as,



muraliganesh92@GaneshLinux:~/Downloads$ ls

**am335x-debian-13.2-base-v6.12-armhf-2025-12-08-4gb.img**



if Partitions are available in sd card and if we want to **erase the partitions and Filesystem signatures**



**sudo wipefs -a /dev/sdb**



E -> Use the below command to copy the img file into sd card.



sudo dd if=<image>.img of=/dev/sdX i 1M status=progress conv=fsync



The actual command shall be,



**sudo dd if=am335x-debian-13.2-base-v6.12-armhf-2025-12-08-4gb.img of=/dev/sdb bs=1M status=progress conv=fsync**



sync



after this, the command, lsblk shall have the output automatically as,



sdb      8:16   1 14.9G  0 disk

├─sdb1   8:17   1  100M  0 part  (boot partition, FAT)

└─sdb2   8:18   1  3.8G  0 part  (root filesystem)



sudo fdisk -l /dev/sdb



Device     Boot   Start     End Sectors  Size Id Type

/dev/sdb1  \*       8192   81919   73728   36M  c W95 FAT32 (LBA)

/dev/sdb2         81920 1130495 1048576  512M 82 Linux swap / Solaris

/dev/sdb3       1130496 7372799 6242304    3G 83 Linux





//The sd card will automatically be partitioned into 3 spaces.



**sudo eject /dev/sdX**



**and boot the board using sd card by pressing the button.**



Now to ssh into beaglebone,



use,



**ssh debian@192.168.7.2**



if you get a warning like below,



@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED! @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@



the write the below command,



**ssh-keygen -f "/home/muraliganesh92/.ssh/known\_hosts" -R "192.168.7.2"**



then again give



**ssh debian@192.168.7.2**





**------------------------------------------------------------------------------------------------------------------------------------------**

**///////////////////////////////////////////////////////Build root Working/////////////////////////////////////////////////////////////////**

**------------------------------------------------------------------------------------------------------------------------------------------**



**// Install Buildroot latest**

2\. git clone https://github.com/buildroot/buildroot.git

   cd buildroot

   make beaglebone\_defconfig                                 **# to find other configurations go to path, buildroot/configs/**

   make menuconfig   					     **# optional customization**

(This gives options like, Target options, Toolchain, Build options, Kernel Target Packages, Filesystem Iages, bootloaders, host utilities, Legacy config options)



TargetOptions -> Target Architecture -> ARM (Little Endian)

 	      -> Target Architecture Variant -> cortex-A8



Toolchain      -> Bootlin toolchains





System Configuration -> System hostname

 		     -> Root Password







Target packages -> Hardware Handling -> i2c tools, minicom, picocom, spi-tools



Target packages -> Networking applications -> can-utils, dropbear, ifupdown scripts, iproute2, lrzsz







   make linux-menuconfig				     **# options for addition of drivers**

(This gives options like, Bus support, device drivers, networking support, cryptographic apis etc for adding drivers)



 





**a) To enable I2c,**



-> Device Drivers -> I2C Support -> \[ \* ] I2C support \[ \* ] I2C device interface



 					 -> I2C Hardware Bus support -> <\*> TI BBPOM / OMAP I2C adapter



**b) To enable Uart,**



 	-> Device Drivers -> Character devices ->  \[ \* ] Serial drivers -> \[ \* ] Serial drivers  \[ \* ] Console on serial port

 									-> 8250/16550 and compatible serial support -> \[ \* ] 8250/16550 and compatible serial support \[ \* ] OMAP8250 support <\*> Support for OMAP internal UART (8250 based driver) <\*> Devicetree based probing for 8250 ports Replace ttyO with ttyS enabled





**c) To enable SPI,**



-> Device Drivers -> SPI Support -> \[ \* ] SPI support \[ \* ] User mode SPI device driver support (spidev) <\*> McSPI driver for OMAP







**d) To enable CAN,**



-> Networking support -> CAN bus subsystem support -> \[ \* ] CAN bus subsystem support \[ \* ] Raw CAN Protocol (raw access with CAN-ID filtering) \[ \* ] Broadcast Manager CAN Protocol (ISO 11898-1)





 	-> Device Drivers  ---> Network device support  ---> CAN Device Drivers  ---> CAN device drivers with Netlink support  ---> \[\*] Bosch C\_CAN/D\_CAN devices

 															 		--->  \[\*] Generic Platform Bus based C\_CAN/D\_CAN driver



**e) Press Esc until <save> appears, and press enter for .config filename to be saved. -> exit**







make   ---- To build the Linux distribution.



now if we want, we can again change configs and give make linux-rebuild, if we change the drivers alone.



but at the end running make will build every changes.





**f) Navigate to ~/Buildroot/output/build/linux-6.18.1/arch/arm/boot/dts/ti/omap** and executing am335x\*.dts -> provides the list of dts files.



code am335x-boneblack.dts



**right below**



\&baseboard\_eeprom {

 	vcc-supply = <\&ldo4\_reg>;

};



**ensure below modifications are added,**





\&am33xx\_pinmux {

 	/\* UART4 Pins: P9.11 (rx) and P9.13 (tx) \*/

 	uart4\_pins: pinmux\_uart4\_pins {

 		pinctrl-single,pins = <

 			AM33XX\_PADCONF(AM335X\_PIN\_GPMC\_AD13, PIN\_INPUT\_PULLUP, MUX\_MODE6) /\* P9.11 \*/

 			AM33XX\_PADCONF(AM335X\_PIN\_GPMC\_AD12, PIN\_OUTPUT\_PULLDOWN, MUX\_MODE6) /\* P9.13 \*/

 		>;

 	};



 	/\* I2C2 Pins: P9.19 (sda) and P9.20 (scl) \*/

 	i2c2\_pins\_custom: pinmux\_i2c2\_pins\_custom {

        pinctrl-single,pins = <

            AM33XX\_PADCONF(AM335X\_PIN\_UART1\_RTSN, PIN\_INPUT\_PULLUP, MUX\_MODE3) /\* P9.19 \*/

            AM33XX\_PADCONF(AM335X\_PIN\_UART1\_CTSN, PIN\_INPUT\_PULLUP, MUX\_MODE3) /\* P9.20 \*/

        >;

    };



 	/\* SPI1 Pins: P9.28 (cs), P9.29 (miso), P9.30 (mosi), P9.31 (sck) \*/

 	spi1\_pins: pinmux\_spi1\_pins {

 		pinctrl-single,pins = <

 			AM33XX\_PADCONF(AM335X\_PIN\_MCASP0\_ACLKX, PIN\_INPUT\_PULLUP, MUX\_MODE3) /\* P9.31 sck \*/

 			AM33XX\_PADCONF(AM335X\_PIN\_MCASP0\_FSX, PIN\_INPUT\_PULLUP, MUX\_MODE3)   /\* P9.29 miso \*/

 			AM33XX\_PADCONF(AM335X\_PIN\_MCASP0\_AXR0, PIN\_INPUT\_PULLUP, MUX\_MODE3)  /\* P9.30 mosi \*/

 			AM33XX\_PADCONF(AM335X\_PIN\_MCASP0\_AHCLKR, PIN\_INPUT\_PULLUP, MUX\_MODE3) /\* P9.28 cs \*/

 		>;

 	};



 	/\* CAN1 Pins: P9.24 (rx) and P9.26 (tx) \*/

 	dcan1\_pins: pinmux\_dcan1\_pins {

 		pinctrl-single,pins = <

 			AM33XX\_PADCONF(AM335X\_PIN\_UART1\_RXD, PIN\_INPUT\_PULLUP, MUX\_MODE2) /\* P9.26 rx \*/

 			AM33XX\_PADCONF(AM335X\_PIN\_UART1\_TXD, PIN\_OUTPUT\_PULLUP, MUX\_MODE2) /\* P9.24 tx \*/

 		>;

 	};

};



\&lcdc {

    status = "disabled";

};





\&mcasp0 {

 	status = "disabled";

};







\&uart4 {

 	pinctrl-names = "default";

 	pinctrl-0 = <\&uart4\_pins>;

 	status = "okay";

};



\&i2c2 {

 	pinctrl-names = "default";

 	pinctrl-0 = <\&i2c2\_pins\_custom>;

 	status = "okay";

 	clock-frequency = <100000>;

};



\&spi1 {

 	pinctrl-names = "default";

 	pinctrl-0 = <\&spi1\_pins>;

 	status = "okay";



 	spidev@0 {

 		compatible = "rohm,dh2228fv"; /\* This string tells kernel to use spidev driver \*/

 		spi-max-frequency = <24000000>;

 		reg = <0>;

 	};

};



\&dcan1 {

 	pinctrl-names = "default";

 	pinctrl-0 = <\&dcan1\_pins>;

 	status = "okay";

};



**////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////**

**/////////////////////////////////////////////////////////////non working code///////////////////////////////////////////////////////////**

**////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////**



\&am33xx\_pinmux {

    dcan1\_pins: dcan1\_pins {

        pinctrl-single,pins = <

            0x190 0x12   /\* spi1\_sclk → dcan1\_tx (P9\_31) \*/

            0x194 0x32   /\* spi1\_d0   → dcan1\_rx (P9\_29) \*/

        >;

    };

};



\&am33xx\_pinmux {

    spi0\_pins: spi0\_pins {

        pinctrl-single,pins = <

            0x150 0x30  /\* P9\_22: spi0\_sclk | Mode 0, Pullup, Input \*/

            0x154 0x30  /\* P9\_21: spi0\_d0   | Mode 0, Pullup, Input \*/

            0x158 0x10  /\* P9\_18: spi0\_d1   | Mode 0, Output \*/

            0x15c 0x10  /\* P9\_17: spi0\_cs0  | Mode 0, Output \*/

        >;

    };

};





\&spi0 {

    status = "okay";

    pinctrl-names = "default";

    pinctrl-0 = <\&spi0\_pins>;



    /\* Ensure #address-cells and #size-cells are here \*/

    #address-cells = <1>;

    #size-cells = <0>;



    /\* Generic SPI device for testing \*/

    testspi@0 {

        compatible = "rohm,dh2228fv", "linux,spidev";

        reg = <0>;

        spi-max-frequency = <24000000>;

    };

};







\&i2c2 {

    status = "okay";

    clock-frequency = <100000>;

};



\&uart1 {

    status = "okay";

};





\&dcan1 {

    status = "okay";



    pinctrl-names = "default";

    pinctrl-0 = <\&dcan1\_pins>;



    can-clock-frequency = <24000000>;



    clocks = <\&l4ls\_gclk>;

    clock-names = "fck";

};





//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////



**After this, give make linux-rebuild**



2.1. To check if the generated files are correct do the below.



a) **cd ~/buildroot/output/images**

b) **ls -lh** will give output as below.



**MLO**

**u-boot.img**

**zImage**

**am335x-boneblack.dtb**



Now verify all four:

**file MLO**

**file u-boot.img**

**file zImage**

**file am335x-boneblack.dtb**



Expected:



**MLO → ARM bootloader / SPL**

**u-boot.img → u-boot legacy uImage**

**zImage → Linux kernel**

**\*.dtb → Device Tree Blob**



3\. if Partitions are available in sd card and if we want to **erase the partitions and Filesystem signatures**



sudo wipefs -a /dev/sdb



4\. **To create partitions:**



sudo fdisk /dev/sdb



inside fdisk we do the following.





**muraliganesh92@GaneshLinux:~$ lsblk -l**

NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS

loop0    7:0    0  73.9M  1 loop /snap/core22/2216

loop1    7:1    0    74M  1 loop /snap/core22/2163

loop2    7:2    0     4K  1 loop /snap/bare/5

loop3    7:3    0 249.1M  1 loop /snap/firefox/7177

loop4    7:4    0 250.1M  1 loop /snap/firefox/7298

loop5    7:5    0  11.1M  1 loop /snap/firmware-updater/167

loop6    7:6    0  18.5M  1 loop /snap/firmware-updater/210

loop7    7:7    0   516M  1 loop /snap/gnome-42-2204/202

loop8    7:8    0  91.7M  1 loop /snap/gtk-common-themes/1535

loop9    7:9    0  10.8M  1 loop /snap/snap-store/1270

loop10   7:10   0  50.9M  1 loop /snap/snapd/25577

loop11   7:11   0  48.1M  1 loop /snap/snapd/25935

loop12   7:12   0   576K  1 loop /snap/snapd-desktop-integration/315

sda      8:0    0    40G  0 disk

sda1     8:1    0     1M  0 part

sda2     8:2    0    25G  0 part /

sdb      8:16   1  14.8G  0 disk

sr0     11:0    1  1024M  0 rom





**muraliganesh92@GaneshLinux:~$ sudo fdisk /dev/sdb**



Welcome to fdisk (util-linux 2.39.3).

Changes will remain in memory only, until you decide to write them.

Be careful before using the write command.



Device does not contain a recognized partition table.

Created a new DOS (MBR) disklabel with disk identifier 0xe32248df.



Command (m for help): m



Help:



  DOS (MBR)

   a   toggle a bootable flag

   b   edit nested BSD disklabel

   c   toggle the dos compatibility flag



  Generic

   **d   delete a partition**

   F   list free unpartitioned space

   l   list known partition types

   **n   add a new partition**

   p   print the partition table

   **t   change a partition type**

   v   verify the partition table

   i   print information about a partition



  Misc

   m   print this menu

   u   change display/entry units

   x   extra functionality (experts only)



  Script

   I   load disk layout from sfdisk script file

   O   dump disk layout to sfdisk script file



  Save \& Exit

   w   write table to disk and exit

   q   quit without saving changes



  Create a new label

   g   create a new empty GPT partition table

   G   create a new empty SGI (IRIX) partition table

   o   create a new empty MBR (DOS) partition table

   s   create a new empty Sun partition table





**Command (m for help): n**

Partition type

   p   primary (0 primary, 0 extended, 4 free)

   e   extended (container for logical partitions)

**Select (default p): p**

**Partition number (1-4, default 1): 1**

First sector (2048-31116287, default 2048):  **#Pressing Enter makes to select the default first memory**

Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-31116287, default 31116287): **+200M   #Providing size**



Created a new partition 1 of type 'Linux' and of size 200 MiB.

Partition #1 contains a vfat signature.



**Do you want to remove the signature? \[Y]es/\[N]o: Y**



The signature will be removed by a write command.



**Command (m for help): t**

**Selected partition 1**

**Hex code or alias (type L to list all): c**

Changed type of partition 'Linux' to 'W95 FAT32 (LBA)'. **#changing to fat32 type**



**Command (m for help): n**

Partition type

   p   primary (1 primary, 0 extended, 3 free)

   e   extended (container for logical partitions)

**Select (default p): p**

**Partition number (2-4, default 2): 2**

First sector (411648-31116287, default 411648): **#Pressing Enter makes to select the default first memory address**

Last sector, +/-sectors or +/-size{K,M,G,T,P} (411648-31116287, default 31116287):  **#Pressing Enter makes to select the default last memory address**



Created a new partition 2 of type 'Linux' and of size 14.6 GiB.

Partition #2 contains a ext4 signature.



**Do you want to remove the signature? \[Y]es/\[N]o: Y**



The signature will be removed by a write command.



**Command (m for help): w**



The partition table has been altered.

Calling ioctl() to re-read partition table.

Syncing disks.



muraliganesh92@GaneshLinux:~$ lsblk

NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS

loop0    7:0    0  73.9M  1 loop /snap/core22/2216

loop1    7:1    0    74M  1 loop /snap/core22/2163

loop2    7:2    0     4K  1 loop /snap/bare/5

loop3    7:3    0 249.1M  1 loop /snap/firefox/7177

loop4    7:4    0 250.1M  1 loop /snap/firefox/7298

loop5    7:5    0  11.1M  1 loop /snap/firmware-updater/167

loop6    7:6    0  18.5M  1 loop /snap/firmware-updater/210

loop7    7:7    0   516M  1 loop /snap/gnome-42-2204/202

loop8    7:8    0  91.7M  1 loop /snap/gtk-common-themes/1535

loop9    7:9    0  10.8M  1 loop /snap/snap-store/1270

loop10   7:10   0  50.9M  1 loop /snap/snapd/25577

loop11   7:11   0  48.1M  1 loop /snap/snapd/25935

loop12   7:12   0   576K  1 loop /snap/snapd-desktop-integration/315

sda      8:0    0    40G  0 disk

├─sda1   8:1    0     1M  0 part

└─sda2   8:2    0    25G  0 part /

sdb      8:16   1  14.8G  0 disk

├─sdb1   8:17   1   200M  0 part

└─sdb2   8:18   1  14.6G  0 part

sr0     11:0    1  1024M  0 rom

muraliganesh92@GaneshLinux:~$



5\. To check if partitions are available, do,



**lsblk -f**



5.1. To check the partitions type.



**sudo blkid /dev/sdb1**

**sudo blkid /dev/sdb2**



6\. If I insert my sd card, and if I am getting something like this,





sdb

  |\_sdb1 8:17 1 36M 0 part /media/muraliganesh92/40128e5A

  |\_sdb2 8:18 1 512M 0 part

  |\_sdb3 8 :19 1 14.3G 0 part /media/muraliganesh92/405D-AD00



then we must unmount the sd card first.

**sudo umount /media/muraliganesh92/40128e5A**

**sudo umount /media/muraliganesh92/405D-AD00**



Then old partitions can be deleted using the command,



sudo wipefs -a /dev/sdb



and new partitions can be created as per step 4.



7\. To format the partitions below is the commands.



**sudo mkfs.vfat -F 32 /dev/sdb1**

**sudo mkfs.ext4 /dev/sdb2**



8\. Create Mounting Points:



**sudo mkdir -p /mnt/boot**

**sudo mkdir -p /mnt/rootfs**



9\. Mount manually (correct way)

**sudo mount /dev/sdb1 /mnt/boot             #Ensure the space is available between sdb1 and /mnt**

**sudo mount /dev/sdb2 /mnt/rootfs**



10\. Copy the files to the locations.



**sudo cp MLO /mnt/boot/**

**sudo cp u-boot.img /mnt/boot/**

**sudo cp zImage /mnt/boot/**

**sudo cp am335x-boneblack.dtb /mnt/boot/**

**sync**



**sudo tar -xvf rootfs.tar -C /mnt/rootfs**

**sync**



10.1 Now check if the sd card partition is in bootable format.



**sudo fdisk -l /dev/sd1**



**Device     Boot Start    End Sectors  Size Id Type**

/dev/sdb1  \*     2048 264191 262144  128M  c W95 FAT32 (LBA)

/dev/sdb2      264192 XXXXX XXXXX   xxG 83 Linux





if \* is not available in Boot for sdb1, then do the following.



**sudo fdisk /dev/sdb**



inside fdisk, do the following,



**a        # toggle boot flag**

**1        # select partition 1**

**w        # write and exit**







11\. To check if the file is copied in the correct order give,



**ls -U /mnt/BOOT   #this shall provide the below output and it is to be ensured the order is maintained.**



MLO  u-boot.img zImage am335x-boneblack.dtb



11\. In case OS is not booting properly,



in the putty terminal, do the follow.



1. After pressing the 2nd button and powering on, click on any key to stop auto booting.



2\. Find out the sd card using the below command,



=> **mmc list**

OMAP SD/MMC: 0 (SD)

OMAP SD/MMC: 1 (eMMC)



3\. check partitions on SD



**=> part list mmc 0**



Partition Map for mmc device 0 -- Partition Type: DOS

Part 	StartSector 	NumSectors	UUID		Type

1 	2048 		409600 		ef599629-01 	0c Boot

2 	411648 		30704640 	ef599629-02 	83



implement the below commands line by line



**=> mmc dev 0**

**=> mmc rescan**

**=> fatls mmc 0:1**

MLO

u-boot.img

zImage

am335x-boneblack.dtb

uEnv.txt



**=> fatload mmc 0:1 0x82000000 zImage**

**=> fatload mmc 0:1 0x88000000 am335x-boneblack.dtb**

**=> setenv bootargs console=ttyS0,115200 root=/dev/mmcblk0p2 rootwait rw**

**=> bootz 0x82000000 - 0x88000000**



---



For uEnv.txt creation :



bootcmd=run mmcboot



mmcboot=\\

mmc dev 0; mmc rescan; \\

fatload mmc 0:1 0x82000000 zImage; \\

fatload mmc 0:1 0x88000000 am335x-boneblack.dtb; \\

setenv bootargs console=ttyS0,115200 root=/dev/mmcblk0p2 rootwait rw; \\

bootz 0x82000000 - 0x88000000



---





**Now to check if drivers are correctly added do the below steps.**



**a) to Check spi**

**ls /sys/class/spi\_master/**

spi0.0



**ls /sys/bus/spi/devices/**

spi0.0





**ls /dev/spidev\***

/dev/spidev0.0



**dmesg | grep -i spi**



**b) to check uart**

**ls /dev/tty\***

/dev/tty0 … /dev/ttyS0 … /dev/ttyS5

**ls /dev/ttyS\***

/dev/ttyS0 /dev/ttyS1 /dev/ttyS2 /dev/ttyS3 /dev/ttyS4 /dev/ttyS5



**c) to check i2c**

**ls /dev/i2c-\***

/dev/i2c-0  /dev/i2c-2





**d) to check CAN**

**ip link show**

3: can0: <NOARP> mtu 16 qdisc noop qlen 10

    link/\[280]

**dmesg | grep -i can**

**find /proc/device-tree -name "\*can\*" | head**

**ip link set can0 up type can bitrate 500000    -> not working as of now, need to check.**



**------------------------------------------------------------------------------------------------------------------------------------------**

**//////////////////////////////////////////////////////////////YOCTO///////////////////////////////////////////////////////////////////////**

**------------------------------------------------------------------------------------------------------------------------------------------**





1. Install vs code



\# Installing VS code using CLI

**sudo apt update**

**sudo apt install snapd**

**sudo snap install --classic code**





2\. Yocto packages



On older version of Ubuntu

**sudo apt install gawk wget git diffstat unzip texinfo gcc build-essential chrpath socat cpio python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev pylint3 xterm python3-subunit mesa-common-dev zstd liblz4-tool**



On newer version of Ubuntu (20.04+, 22.04, 24.04)

**sudo apt install \\**

**gawk wget git diffstat unzip texinfo gcc build-essential chrpath socat \\**

**cpio python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping \\**

**python3-git python3-jinja2 libegl1 libsdl1.2-dev pylint xterm \\**

**python3-subunit mesa-common-dev zstd liblz4-tool**





3\. Clone Poky(reference distribution)



**mkdir yocto-project**

**cd yocto-project**

**git clone git://git.yoctoproject.org/poky -b kirkstone**

(or)

**git clone git://git.yoctoproject.org/poky**



**cd poky**

**git branch -a**

**git checkout -t origin/kirkstone -b my-kirkstone**



Branch 'my-kirkstone' set up to track remote branch 'kirkstone' from 'origin'.

Switched to a new branch 'my-kirkstone'



**git pull**



4\. Initialize Build Environment



**cd poky**

**source oe-init-build-env**



the directory is not changed to build. In this runt the command,



**bitbake core-image-full-cmdline**



**------------------------------------------------------------------------------------------------------------------------------------------**

**Important files to modify.**



1. **Layers.conf**

yocto->poky->meta-poky->conf->layer.conf

**MACHINE** ?= "beaglebone-yocto"

**DISTRO** ?= "poky"

**PACKAGE\_CLASSES** ?= "package\_rpm"

**EXTRA\_IMAGE\_FEATURES** ?=  "debug-tweaks"  add additional features like, "debug-tweaks dbg-pkgs src-pkgs" etc.

**USER\_CLASSES** ?= "buildstats"

**PATCHRESOLVE** = "noop"      #it can be user as well. noop -> stop build if any errors, user -> stops, makes users to correct it and then again run.

**BB\_DISKMON\_DIRS** ??= "\\STOPTASK, ${TEMPDIR}, 1G, 100K" … etc, -> monitors disk space

**PACKAGECONFIG**:append:pn-qemu-system-native - " sdl"

**CONF\_VERSION** = "2"

**RM\_OLD\_IMAGE** = "1"  #delets old packages and creates new ones.

**INHERIT** += "rm\_work"





\#========== MACHINE SELECTION ==========

MACHINE = "beaglebone-yocto"



\#========== DISTRO SELECTION ==========

DISTRO = "poky"



\#========== PACKAGE MANAGEMENT ==========

\# Change package format: deb, ipk, or rpm

PACKAGE\_CLASSES = "package\_rpm"



\#========== EXTRA IMAGE FEATURES ==========

\# Add debugging and development tools

EXTRA\_IMAGE\_FEATURES += "dbg-pkgs src-pkgs dev-pkgs debug-tweaks"

EXTRA\_IMAGE\_FEATURES += "tools-sdk tools-debug tools-profile"

EXTRA\_IMAGE\_FEATURES += "ptest-pkgs"



\#========== USER CLASSES ==========

\# Add build statistics and other features

USER\_CLASSES = "buildstats buildstats-summary image-prelink"



\#========== PREFERRED VERSIONS ==========

\# Override recipe versions

PREFERRED\_VERSION\_linux-yocto = "6.1%"

PREFERRED\_VERSION\_u-boot = "2023.01%"

PREFERRED\_VERSION\_gcc = "13%"



\#========== IMAGE FSTYPES ==========

\# Change output image formats

IMAGE\_FSTYPES = "tar.gz ext4 wic wic.gz"



\#========== BUILD PARALLELIZATION ==========

\# Speed up builds

BB\_NUMBER\_THREADS = "8"

PARALLEL\_MAKE = "-j 8"



\#========== OPTIMIZATION FLAGS ==========

\# Modify compiler optimization

EXTRA\_IMAGE\_FEATURES += "read-only-rootfs"

CFLAGS = "-O2 -pipe"

CXXFLAGS = "-O2 -pipe"



\#========== BITBAKE CONFIGURATION ==========

\# Enable hash equivalence for better sstate reuse

BB\_HASHSERVE = "auto"

BB\_SIGNATURE\_HANDLER = "OEEquivHash"



\# Keep server in memory for faster builds

BB\_SERVER\_TIMEOUT = "60"



\#========== DISK MONITORING ==========

\# Adjust disk space requirements

BB\_DISKMON\_DIRS ??= "\\

    STOPTASKS,${TMPDIR},1G,100K \\

    STOPTASKS,${DL\_DIR},1G,100K \\

    STOPTASKS,${SSTATE\_DIR},1G,100K \\

    STOPTASKS,/tmp,100M,100K \\

    HALT,${TMPDIR},100M,1K \\

    HALT,${DL\_DIR},100M,1K \\

    HALT,${SSTATE\_DIR},100M,1K \\

    HALT,/tmp,10M,1K"



\#========== DOWNLOAD \& CACHING ==========

DL\_DIR ?= "${TOPDIR}/downloads"

SSTATE\_DIR ?= "${TOPDIR}/sstate-cache"

TMPDIR = "${TOPDIR}/tmp"



\#========== QEMU CONFIGURATION ==========

PACKAGECONFIG:append:pn-qemu-system-native = " sdl"

\#PACKAGECONFIG:append:pn-qemu-system-native = " gtk+"



\#========== PACKAGE CONFIGURATIONS ==========

\# Add/remove features from specific packages

PACKAGECONFIG:append:pn-openssl = " fips"

PACKAGECONFIG:remove:pn-gobject-introspection = "tests"



\#========== FEATURE TOGGLES ==========

\# Enable/disable specific distro features

DISTRO\_FEATURES:append = " systemd wifi bluetooth"

DISTRO\_FEATURES:remove = " sysvinit x11"



\#========== KERNEL COMMAND LINE ==========

\# Modify kernel boot parameters

APPEND:append = " quiet loglevel=3"



\#========== INHERIT CLASSES ==========

INHERIT += "rm\_work"

INHERIT += "buildstats"

INHERIT += "image-postproc"



\#========== LICENSE FILTERING ==========

\# Accept specific licenses

LICENSE\_FLAGS\_WHITELIST = "commercial"



\#========== SECURITY SETTINGS ==========

\# Enable security features

EXTRA\_IMAGE\_FEATURES += "package-management"



\#========== ROOT PASSWORD ==========

\# Set root password (debug only)

\#EXTRA\_IMAGE\_FEATURES += "debug-tweaks"



\#========== BUILD HISTORY ==========

\# Track build changes

BUILDHISTORY = "1"

BUILDHISTORY\_COMMIT = "1"



\#========== SSTATE MIRRORS ==========

\# Use pre-built artifacts

\#SSTATE\_MIRRORS ?= "\\

\#file://.\* https://someserver.tld/share/sstate/PATH;downloadfilename=PATH \\

\#file://.\* file:///some/local/dir/sstate/PATH"





**2. Machine.conf**

yocto->poky->meta-poky->meta-yocto-bsp->conf->machine->beaglebone-yocto.conf



\# Change the default kernel

PREFERRED\_PROVIDER\_virtual/kernel = "linux-yocto"

PREFERRED\_VERSION\_linux-yocto = "6.1%"



\# Change U-Boot version

PREFERRED\_VERSION\_u-boot = "2023.01"



\# Modify kernel device tree

KERNEL\_DEVICETREE = "am335x-boneblack.dtb am335x-bone.dtb"



\# Change kernel command line

KERNEL\_EXTRA\_ARGS = "LOADADDR=0x80008000"



\# Modify machine features (hardware capabilities)

MACHINE\_FEATURES = "usbgadget usbhost vfat ext2 alsa ethernet wifi bluetooth"



\# Change serial console settings

SERIAL\_CONSOLE = "115200 ttyO0"



\# Modify machine overrides for conditional settings

MACHINEOVERRIDES =. "beaglebone:"



\# Change default image type

IMAGE\_FSTYPES = "tar.gz ext4 wic"



\# Modify bootloader configuration

UBOOT\_MACHINE = "am335x\_evm\_defconfig"



\# Change kernel build options

KERNEL\_IMAGETYPE = "zImage"



\# Tune CPU specific optimizations

DEFAULTTUNE = "cortexa8hf-neon"







**3. BBlayer.conf**

yocto->poky->meta-poky->conf->bblayers.conf









---



Writing a Yocto Recipe from Scratch



\## Key Terminology



Recipe - A .bb file that contains instructions on how to build a software package. It defines where to get the source code, how to configure it, how to build it, and how to package it.



Layer - A collection of recipes and metadata organized in a specific directory structure. Allows modular organization of Yocto projects.



BitBake - The build engine that executes recipes and manages dependencies.



Variable - A named value that can be used throughout the recipe (e.g., SRC\_URI, AUTHOR).



Task - A function executed during the build process (e.g., do\_fetch, do\_compile, do\_install).



Metadata - Recipe files and configuration that describe how to build packages.



---



\## Basic Recipe Structure



A minimal recipe (myapp\_1.0.bb) contains:



\# Metadata

SUMMARY = "Brief description"

DESCRIPTION = "Longer description of what the package does"

AUTHOR = "Your Name"

HOMEPAGE = "https://example.com"

LICENSE = "MIT"



\# Source code location

SRC\_URI = "https://github.com/user/myapp/archive/v1.0.tar.gz"

SRC\_URI\[sha256sum] = "abc123..."



\# Dependencies

DEPENDS = "zlib openssl"



\# Build configuration

inherit autotools



---



\## Essential Variables Explained



| Variable 		| Purpose 			| Example 					|

|-----------------------|-------------------------------|-----------------------------------------------|

| SUMMARY 		| One-line description 		| "A lightweight HTTP server" 			|

| DESCRIPTION 		| Detailed description 		| "MyApp is a fast HTTP server..." 		|

| LICENSE 		| License type 			| "Apache-2.0", "MIT", "GPL-2.0-only" 		|

| AUTHOR 		| Package author 		| "John Doe" 					|

| HOMEPAGE 		| Project URL 			| "https://github.com/user/project" 		|

| SRC\_URI 		| Source download location 	| "https://example.com/file.tar.gz" 		|

| SRC\_URI\[sha256sum]	| Checksum for integrity 	| "abc123def456..." 				|

| S 			| Build directory path 		| "${WORKDIR}/myapp-1.0" 			|

| D 			| Installation destination 	| "${WORKDIR}/image" (auto-set) 		|

| DEPENDS 		| Build-time dependencies 	| "openssl zlib" 				|

| RDEPENDS 		| Runtime dependencies 		| "bash libssl" 				|

| PV 			| Package version 		| "1.0" (extracted from filename) 		|

| PN 			| Package name 			| "myapp" (extracted from filename) 		|







---



\## Build Tasks Explained



These are executed in order:



1\. do\_fetch - Downloads source code from SRC\_URI

2\. do\_unpack - Extracts the source tarball

3\. do\_patch - Applies patches from \*.patch files

4\. do\_configure - Runs ./configure script (if using autotools)

5\. do\_compile - Compiles source code (make)

6\. do\_install - Installs files to ${D} directory

7\. do\_package - Creates deployable packages (.deb, .rpm, etc.)



---









\## Complete Example Recipe



Here's a realistic recipe for a simple C application:





\# recipes-apps/myapp/myapp\_1.0.bb



SUMMARY = "MyApp - A Simple File Processor"

DESCRIPTION = "MyApp is a command-line tool for processing configuration files"

HOMEPAGE = "https://github.com/mycompany/myapp"

AUTHOR = "John Doe [john@example.com](mailto:john@example.com)"

LICENSE = "MIT"



\# Inherit standard build system (autotools in this case)

inherit autotools



\# Source code location and checksum

SRC\_URI = "https://github.com/mycompany/myapp/archive/v${PV}.tar.gz"

SRC\_URI\[sha256sum] = "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z6"



\# Build-time dependencies (needed to compile)

DEPENDS = "openssl zlib"



\# Runtime dependencies (needed to run)

RDEPENDS:${PN} = "bash libc6"



\# Where the source code gets unpacked

S = "${WORKDIR}/myapp-${PV}"



\# Add custom patches

SRC\_URI += "file://fix-gcc-warning.patch"



\# Custom installation task

do\_install() {

    install -d ${D}${bindir}

    install -m 0755 ${B}/myapp ${D}${bindir}/

 

    install -d ${D}${sysconfdir}

    install -m 0644 ${S}/myapp.conf ${D}${sysconfdir}/

}



\# Custom tasks can also be added

do\_compile:prepend() {

    echo "Starting compilation of MyApp..."

}







\## Directory Structure





meta-mylayer/

├── conf/

│   └── layer.conf

├── recipes-apps/

│   └── myapp/

│       ├── myapp\_1.0.bb

│       ├── files/

│       │   └── fix-gcc-warning.patch

│       └── myapp/

│           └── myapp.inc (optional: shared variables)

└── recipes-core/

    └── (other recipe categories)





\## Key Conventions



\- Recipe filename format: name\_version.bb (e.g., httpd\_2.4.48.bb)

\- Version extracted from filename: Yocto automatically sets PN (name) and PV (version)

\- Categories: Recipes organized by function (recipes-apps, recipes-core, recipes-kernel, etc.)

\- Patches: Place in files/ subdirectory, reference with  prefix



---



\## Build System Inheritance



Use inherit to leverage pre-built build logic:



\- inherit autotools - For GNU autotools projects (configure, make)

\- inherit cmake - For CMake projects

\- inherit python - For Python packages

\- inherit native - For host tools (not target tools)



---



**------------------------------------------------------------------------------------------------------------------------------------------**







COMPLETE LIST: What Can Be Customized Using Yocto



I’ll break this down layer by layer, exactly how a real OS is structured.



**1. Hardware \& Board Level (Machine Configuration)**



You control how Linux talks to hardware.



✅ CPU \& Architecture



ARM / ARM64 / x86 / x86\_64 / RISC-V / MIPS

Endianness (little/big)

ABI (EABI, hard-float, soft-float)

CPU tuning (Cortex-A8, A53, A72, etc.)

FPU / NEON support



✅ Board Definition (MACHINE)



SoC family

Board name

RAM size assumptions

Flash type (NAND / eMMC / SD)

Boot media

Device tree selection



**2. Bootloader (First Code That Runs)**



Yocto can build, configure, and install:



U-Boot

Barebox

GRUB (x86)

Custom bootloaders



You can customize:



Boot delay

Boot commands

Default kernel

Rootfs location

Secure boot



Environment variables



Splash screen

Fast boot / silent boot



**3. Linux Kernel (100% Custom)**



You decide everything about the kernel.



✅ Kernel Version



Mainline

LTS

Vendor kernel

Custom fork



✅ Kernel Configuration



Enable/disable drivers

Filesystems (ext4, squashfs, ubifs, nfs)

Networking stacks

USB / PCI / SPI / I2C / UART

Power management

Debug options

Preemption model

Tick rate (HZ)

Security modules (SELinux, AppArmor)



✅ Device Tree



GPIO mapping

LEDs

Buttons

Displays

Touchscreens

Sensors

Audio codecs

Custom peripherals



**4. Root Filesystem (This Is the OS)**



Yocto gives full control over the rootfs.



✅ Base System



BusyBox or full GNU coreutils

System layout (/bin, /sbin, /usr, /lib)

Init system (SysVinit / systemd)



✅ Filesystem Type



ext4

squashfs (read-only)

ubifs

initramfs

cpio

tar.gz

wic disk images



**5. Init System \& Boot Process**



You control how the system starts.



Choose:



SysVinit

systemd



BusyBox init

Custom init

Customize:



Boot order

Services started

Parallel boot

Boot time optimization

Emergency mode

Rescue shell

Console login behavior



**6. Packages \& Applications**



You choose exactly what software exists.



Examples:



SSH (openssh / dropbear)

Networking tools

Editors (vi, nano)

Debug tools (strace, gdb)

Python / C / C++

Qt / GTK

Web servers

Databases

Custom proprietary apps



You control:



Version

Compile options

Dependencies

Runtime size

License compliance



**7. Programming Languages \& Runtimes**



You can include or exclude:



C / C++

Python (full / minimal)

Java

Node.js

Go

Rust

Lua

Shell scripting



And customize:



Standard libraries

Package managers

Bytecode support

Debug symbols



**8. Users, Groups \& Permissions**



Yocto lets you define the OS users.



Root password (or disable it)

Create custom users

UID/GID control

sudo access

Home directory contents

Default shell

Login policies



**9. Services \& Daemons**



You fully control background processes.



Examples:



SSH

Network manager

Bluetooth

Wi-Fi

Time sync

Logging services

Custom daemons



You decide:



Enabled or disabled

Startup order

Restart policy

Resource limits



**10. Security \& Hardening**



This is huge in Yocto.



You can customize:



Secure boot

Signed images

Read-only rootfs

SELinux / AppArmor



Kernel hardening



Password policies

Firewall rules

Disable unused interfaces

Debug access removal



**11. Package Management \& Updates**



Choose:



rpm

deb

ipk

No package manager at all



Customize:



OTA update mechanism

A/B partitions

Image rollback

Delta updates

Update server integration



**12. Graphics \& UI Stack**



You decide if the OS even has a display.



Options:



No GUI (headless)

Framebuffer only

Wayland

X11

Qt Embedded

Custom UI



You control:



Resolution

Touch support

GPU drivers

Splash screen

Fonts

Themes



**13. Networking Stack**



Customize:



IPv4 / IPv6

DHCP / static IP

Ethernet / Wi-Fi / LTE

Bluetooth

CAN bus

Firewall

VPN



**14. Storage \& Partitioning**



Yocto controls how storage is laid out.



Partition scheme

Boot partition

Rootfs partition

Data partition

Read-only vs read-write

Overlay filesystem



**15. Logging \& Debugging**



Choose:



syslog / journald

Debug symbols

Core dumps

Crash handlers

Remote debugging

Serial console behavior



**16. Power \& Performance**



You can tune:



CPU governors

Frequency scaling

Suspend / resume

Thermal limits

Boot time

Memory footprint

Background services



**17. Licensing \& Compliance**



Yocto tracks:



Open-source licenses

Proprietary blobs

License manifests

Compliance reports



**18. Build System Itself**



You can customize:



Compiler (gcc version)

Optimization flags

Debug vs release

Reproducible builds

CI integration

Build caching

SDK generation

