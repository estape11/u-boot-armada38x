# Uboot binaries to support Mender for the TS-7800-V2 board
## Files
- u-boot.imx -> patched bootloader
- fw_env.config -> env address configuration
- fw_printenv -> bootloader env tool
- fw_setenv -> bootloader set env tool

## Install the new uboot
- Copy the `u-boot-spl.kwb` file to the root of a SD card
- Boot the board and login to the root user
- Run the following commands
```bash
cd /
echo 0 > /sys/block/mmcblk0boot0/force_ro
dd if=u-boot-spl.kwb of=/dev/mmcblk0boot0
sync
echo 1 > /sys/block/mmcblk0boot0/force_ro
```

- Restart and enter the uboot shell again
- Run the following commands
```bash
env default a
saveenv
```

## Install the env tools
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

## Test
Test if the `fw_printenv` tool can access the bootloader enviroment by running
```bash
fw_printenv
```
The output should look similar to:
```
altbootcmd=run mender_altbootcmd; run bootcmd
autoload=no
baudrate=115200
board_id=0xB480
board_model=7800-V2
board_name=TS-7800-V2
board_revision=P1
bootargs=root=/dev/tssdcarda2 rw rootwait console=ttyS0,115200
bootcmd=if test ${jp_sdboot} = 'on';then run sdroot;else run emmcboot;fi;
bootcount=1
bootdelay=0
bootlimit=1
clearenv=mmc dev 0 1; mmc erase 2000 2000; mmc erase 3000 2000;
cmdline_append=console=ttyS0,115200
emmcboot=echo Booting from the eMMC ...;if load mmc 0:1 ${scriptaddr} /boot/boot.ub;then echo Booting from custom /boot/boot.ub;source ${scriptaddr};fi;load mmc 0:1 ${fdt_addr_r} /boot/armada-385-ts7800-v2.dtb;load mmc 0:1 ${kernel_addr_r} /boot/zImage;setenv bootargs r;
eth1addr=e8:1a:58:00:5e:7c
ethact=ethernet@70000
ethaddr=e8:1a:58:00:5e:7c
fdt_addr_r=0x100000
fdt_high=0x10000000
fdtaddr=0x100000
fdtcontroladdr=3fb3e820
fpga_rev=0x2F
initrd_high=0x10000000
jp_sdboot=on
jp_uboot=off
kernel_addr_r=0x800000
loadaddr=0x800000
lola=1
mender_altbootcmd=if test ${mender_boot_part} = 2; then setenv mender_boot_part 3; setenv mender_boot_part_hex 3; else setenv mender_boot_part 2; setenv mender_boot_part_hex 2; fi; setenv upgrade_available 0; saveenv; run mender_setup
mender_boot_kernel_type=bootz
mender_boot_part=2
mender_boot_part_hex=2
mender_boot_part_name=/dev/tssdcarda2
mender_check_saveenv_canary=1
mender_dtb_name=armada-385-ts7800-v2.dtb
mender_kernel_name=zImage
mender_kernel_root=tssdcarda2
mender_kernel_root_name=/dev/tssdcarda2
mender_saveenv_canary=1
mender_setup=if test "${mender_saveenv_canary}" != "1"; then setenv mender_saveenv_canary 1; saveenv; fi; if test "${mender_pre_setup_commands}" != ""; then run mender_pre_setup_commands; fi; if test "${mender_systemd_machine_id}" != ""; then setenv bootargs "systemd.mai
mender_try_to_recover=if test ${upgrade_available} = 1; then reset; fi
mender_uboot_boot=mmc 0:1
mender_uboot_dev=0
mender_uboot_if=mmc
mender_uboot_root=mmc 0:2
mender_uboot_root_name=/dev/tssdcarda2
nfsboot=echo Booting from NFS ...;dhcp;nfs ${fdt_addr_r} ${nfsroot}/boot/armada-385-ts7800-v2.dtb;nfs ${kernel_addr_r} ${nfsroot}/boot/zImage;setenv bootargs root=/dev/nfs rw ip=dhcp nfsroot=${nfsroot} ${cmdline_append};bootz ${kernel_addr_r} - ${fdt_addr_r};
nfsroot=192.168.0.36:/mnt/storage/a38x
preboot=if test ${jp_uboot} = 'on'; then setenv bootdelay -1;run usbprod;else setenv bootdelay 0;fi
pxefile_addr_r=0x300000
ramdisk_addr_r=0x1800000
sataboot=echo Booting from SATA ...;scsi scan;scsi dev ${satadev};part uuid scsi ${satadev}:1 partuuid;if load scsi ${satadev}:1 ${scriptaddr} /boot/boot.ub;then echo Booting from custom /boot/boot.ub;source ${scriptaddr};fi;load scsi ${satadev}:1 ${fdt_addr_r} /boot/ar;
satadev=0
scriptaddr=0x200000
sdroot=echo Booting from the SD Card ...;run mender_setup; load tssdcard 0:2 ${fdt_addr_r} /boot/armada-385-ts7800-v2.dtb;load tssdcard 0:2 ${kernel_addr_r} /boot/zImage;setenv bootargs root=${mender_boot_part_name} rw rootwait ${cmdline_append};bootz ${kernel_addr_r} -;
stderr=serial@12000
stdin=serial@12000
stdout=serial@12000
upgrade_available=0
usbboot=echo Booting from USB ...;usb start;if load usb 0:1 ${scriptaddr} /boot/boot.ub;then echo Booting from custom /boot/boot.ub;source ${scriptaddr};fi;part uuid usb 0:1 partuuid;load usb 0:1 ${fdt_addr_r} /boot/armada-385-ts7800-v2.dtb;load usb 0:1 ${kernel_addr_r};
usbprod=usb start;if usb storage;then echo Checking USB storage for updates;if load usb 0:1 ${scriptaddr} /tsinit.ub;then led 0 on;source ${scriptaddr};led 1 off;exit;fi;fi;
ver=U-Boot 2017.09-g85ba7f9168-dirty (Oct 24 2024 - 17:23:39 -0600)

Environment size: 4767/131067 bytes
```
Now if you set a new variable, this should be saved in the env memory
```bash
fw_setenv test 1234
fw_printenv | grep test
```
It should output:
```bash
test=1234
```