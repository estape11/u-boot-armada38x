
# U-boot dev for TS-7800-V2

## Setting up the enviroment

### Install the dependencies
```bash
sudo apt-get install gcc make gcc-arm-linux-gnueabi binutils-arm-linux-gnueabi device-tree-compiler -y
```

### Clone the project
```
git clone https://github.com/estape11/u-boot-armada38x -b u-boot-2017.09-mender
cd u-boot-armada38x
```

### Download the toolchain
```
wget https://files.embeddedts.com/ftp/ftp/ts-arm-sbc/ts-7800-v2-linux/cross-toolchains/armv7-marvell-linux-uclibcgnueabi-softfp_i686_64K_Dev_20131002.tar.bz2
```

### Extract it
```
mkdir toolchain
tar -xf armv7-marvell-linux-uclibcgnueabi-softfp_i686_64K_Dev_20131002.tar.bz2 -C toolchain/
```

## Set the new enviroment
```
export ARCH=arm
export CROSS_COMPILE=$PWD/toolchain/armv7-marvell-linux-uclibcgnueabi-softfp_i686_64K_Dev_20131002/bin/arm-marvell-linux-uclibcgnueabi-
export CROSS_COMPILE_BH=${CROSS_COMPILE}
```

### Compile
```
make ts7800-v2_defconfig clean all
```

## Stock U-boot env
```
autoload=no
baudrate=115200
bootcmd=if test ${jp_sdboot} = 'on';then run sdroot;else run emmcboot;fi;
bootdelay=3
clearenv=mmc dev 0 1; mmc erase 2000 2000; mmc erase 3000 2000;
cmdline_append=console=ttyS0,115200
emmcboot=echo Booting from the eMMC ...;if load mmc 0:1 ${scriptaddr} /boot/boot.ub;then echo Booting from custom /boot/boot.ub;source ${scriptaddr};fi;load mmc 0:1 ${fdt_addr_r} /boot/armada-385-ts7800-v2.dtb;load mmc 0:1 ${kernel_addr_r} /boot/zImage;setenv bootargs root=/dev/mmcblk0p1 rw rootwait ${cmdline_append};bootz ${kernel_addr_r} - ${fdt_addr_r};
ethact=ethernet@70000
fdt_addr_r=0x100000
fdt_high=0x10000000
fdtaddr=0x100000
initrd_high=0x10000000
kernel_addr_r=0x800000
loadaddr=0x800000
nfsboot=echo Booting from NFS ...;dhcp;nfs ${fdt_addr_r} ${nfsroot}/boot/armada-385-ts7800-v2.dtb;nfs ${kernel_addr_r} ${nfsroot}/boot/zImage;setenv bootargs root=/dev/nfs rw ip=dhcp nfsroot=${nfsroot} ${cmdline_append};bootz ${kernel_addr_r} - ${fdt_addr_r};
nfsroot=192.168.0.36:/mnt/storage/a38x
preboot=if test ${jp_uboot} = 'on'; then setenv bootdelay -1;run usbprod;else setenv bootdelay 0;fi
pxefile_addr_r=0x300000
ramdisk_addr_r=0x1800000
sataboot=echo Booting from SATA ...;scsi scan;scsi dev ${satadev};part uuid scsi ${satadev}:1 partuuid;if load scsi ${satadev}:1 ${scriptaddr} /boot/boot.ub;then echo Booting from custom /boot/boot.ub;source ${scriptaddr};fi;load scsi ${satadev}:1 ${fdt_addr_r} /boot/armada-385-ts7800-v2.dtb;load scsi ${satadev}:1 ${kernel_addr_r} /boot/zImage;setenv bootargs root=PARTUUID=${partuuid} rw rootwait ${cmdline_append};bootz ${kernel_addr_r} - ${fdt_addr_r};
satadev=0
scriptaddr=0x200000
sdroot=echo Booting from the SD Card ...;load tssdcard 0:1 ${fdt_addr_r} /boot/armada-385-ts7800-v2.dtb;load tssdcard 0:1 ${kernel_addr_r} /boot/zImage.nope;setenv bootargs root=/dev/tssdcarda1 rw rootwait ${cmdline_append};bootz ${kernel_addr_r} - ${fdt_addr_r};
usbboot=echo Booting from USB ...;usb start;if load usb 0:1 ${scriptaddr} /boot/boot.ub;then echo Booting from custom /boot/boot.ub;source ${scriptaddr};fi;part uuid usb 0:1 partuuid;load usb 0:1 ${fdt_addr_r} /boot/armada-385-ts7800-v2.dtb;load usb 0:1 ${kernel_addr_r} /boot/zImage;setenv bootargs root=PARTUUID=${partuuid} rw rootwait ${cmdline_append};bootz ${kernel_addr_r} - ${fdt_addr_r};
usbprod=usb start;if usb storage;then echo Checking USB storage for updates;if load usb 0:1 ${scriptaddr} /tsinit.ub;then led 0 on;source ${scriptaddr};led 1 off;exit;fi;fi;

Environment size: 2532/131067 bytes
```

## Development
### Menuconfig
```
sudo apt-get install libncurses-dev
make menuconfig
```

## Mender integration
### Create the config_mender_defines.h
Using the auto-generation script from [meta-mender](https://raw.githubusercontent.com/mendersoftware/meta-mender/kirkstone/meta-mender-core/recipes-bsp/u-boot/u-boot-mender.inc) define the needed variables for Mender to work


## Board related steps
### Flash u-boot
```bash
echo 0 > /sys/block/mmcblk0boot0/force_ro
dd if=u-boot-spl.kwb of=/dev/mmcblk0boot0
sync
echo 1 > /sys/block/mmcblk0boot0/force_ro
```

### Install the uboot env tools
- Copy the files to the SD card of a working rootfs
- Copy the `fw_printenv` file and `fw_setenv` script to the following directory
    ```bash
    cp fw_printenv /usr/bin/
    cp fw_setenv /usr/bin/
    ```
- Copy the configuration
    ```bash
    cp fw_env.config /etc/fw_env.config
    ```

### Create image from rootfs
- Download the image
```bash
wget https://files.embeddedts.com/ftp/ts-arm-sbc/ts-7800-v2-linux/distributions/ts7800v2-debian-bullseye-20220113.tar.xz
```

```bash
# Create the blank image
dd if=/dev/zero of=ts7800v2-debian-bullseye-20220113.img bs=1024k seek=2048 count=0

# Partition table
sudo parted ts7800v2-debian-bullseye-20220113.img mklabel msdos

# Mount the image into a devloop
sudo losetup -fP ts7800v2-debian-bullseye-20220113.img
sudo losetup -a | grep ts7800v2-debian-bullseye-20220113.img

# Create the partition and save the changes
sudo fdisk /dev/<LOOP_DEV>

# Create the file system
sudo mkfs.ext3 /dev/<LOOP_DEV_PART>

# Copy the content into the image
sudo mount /dev/<LOOP_DEV_PART> /mnt
sudo tar --numeric-owner -xJf ts7800v2-debian-bullseye-20220113.tar.xz  -C /mnt/
sync
sudo umount /mnt
sudo losetup -d /dev/<LOOP_DEV>
```

### Mender convert
The following component is needed for the qemu to work with mender-convert
```bash
sudo apt install qemu qemu-user-static
```