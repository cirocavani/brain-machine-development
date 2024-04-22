# How to emulate Raspberry Pi in QEMU

<https://interrupt.memfault.com/blog/emulating-raspberry-pi-in-qemu>

<https://forums.raspberrypi.com/viewtopic.php?p=2207807&sid=1d5a513e231dc5bf7faa3e9ad8a662ef#p2207807>

<https://github.com/raspberrypi/utils/tree/master/dtmerge>

<https://www.raspberrypi.com/software/operating-systems/#raspberry-pi-os-64-bit>

<https://www.qemu.org/docs/master/system/arm/raspi.html>

```sh
mkdir -p rpi-qemu
cd rpi-qemu

brew install xz openssl

curl -kLO https://downloads.raspberrypi.com/raspios_lite_arm64/images/raspios_lite_arm64-2024-03-15/2024-03-15-raspios-bookworm-arm64-lite.img.xz

curl -kLO https://downloads.raspberrypi.com/raspios_lite_arm64/images/raspios_lite_arm64-2024-03-15/2024-03-15-raspios-bookworm-arm64-lite.img.xz.sha256

shasum --check 2024-03-15-raspios-bookworm-arm64-lite.img.xz.sha256
# 2024-03-15-raspios-bookworm-arm64-lite.img.xz: OK

xz -dk 2024-03-15-raspios-bookworm-arm64-lite.img.xz

ls -alh 2024-03-15-raspios-bookworm-arm64-lite.img
# -rw-r--r--  1 ciro.cavani  709999406   2.6G Apr 21 18:04 2024-03-15-raspios-bookworm-arm64-lite.img

hdiutil attach -imagekey diskimage-class=CRawDiskImage -nomount ./2024-03-15-raspios-bookworm-arm64-lite.img
# /dev/disk4              FDisk_partition_scheme
# /dev/disk4s1            Windows_FAT_32
# /dev/disk4s2            Linux

mkdir -p boot
mount -t msdos /dev/disk4s1 ./boot
# Executing: /usr/bin/kmutil load -p /System/Library/Extensions/msdosfs.kext

ls -alh boot
# total 124660
# drwxrwxrwx@ 1 ciro.cavani  staff       4.0K Apr 21 18:53 .
# drwxr-xr-x  5 ciro.cavani  709999406   160B Apr 21 18:46 ..
# drwxrwxrwx  1 ciro.cavani  staff       2.0K Apr 21 18:53 .fseventsd
# -rwxrwxrwx  1 ciro.cavani  staff       1.6K Mar 15 14:00 LICENCE.broadcom
# -rwxrwxrwx  1 ciro.cavani  staff        31K Mar  7 13:51 bcm2710-rpi-2-b.dtb
# -rwxrwxrwx  1 ciro.cavani  staff        33K Mar  7 13:51 bcm2710-rpi-3-b-plus.dtb
# -rwxrwxrwx  1 ciro.cavani  staff        33K Mar  7 13:51 bcm2710-rpi-3-b.dtb
# -rwxrwxrwx  1 ciro.cavani  staff        31K Mar  7 13:51 bcm2710-rpi-cm3.dtb
# -rwxrwxrwx  1 ciro.cavani  staff        32K Mar  7 13:51 bcm2710-rpi-zero-2-w.dtb
# -rwxrwxrwx  1 ciro.cavani  staff        32K Mar  7 13:51 bcm2710-rpi-zero-2.dtb
# -rwxrwxrwx  1 ciro.cavani  staff        54K Mar  7 13:51 bcm2711-rpi-4-b.dtb
# -rwxrwxrwx  1 ciro.cavani  staff        54K Mar  7 13:51 bcm2711-rpi-400.dtb
# -rwxrwxrwx  1 ciro.cavani  staff        37K Mar  7 13:51 bcm2711-rpi-cm4-io.dtb
# -rwxrwxrwx  1 ciro.cavani  staff        54K Mar  7 13:51 bcm2711-rpi-cm4.dtb
# -rwxrwxrwx  1 ciro.cavani  staff        51K Mar  7 13:51 bcm2711-rpi-cm4s.dtb
# -rwxrwxrwx  1 ciro.cavani  staff        76K Mar  7 13:51 bcm2712-rpi-5-b.dtb
# -rwxrwxrwx  1 ciro.cavani  staff        76K Mar  7 13:51 bcm2712-rpi-cm5-cm4io.dtb
# -rwxrwxrwx  1 ciro.cavani  staff        76K Mar  7 13:51 bcm2712-rpi-cm5-cm5io.dtb
# -rwxrwxrwx  1 ciro.cavani  staff        76K Mar  7 13:51 bcm2712d0-rpi-5-b.dtb
# -rwxrwxrwx  1 ciro.cavani  staff        51K Mar 15 14:00 bootcode.bin
# -rwxrwxrwx  1 ciro.cavani  staff       154B Mar 15 14:08 cmdline.txt
# -rwxrwxrwx  1 ciro.cavani  staff       1.2K Mar 15 14:00 config.txt
# -rwxrwxrwx  1 ciro.cavani  staff       7.1K Mar 15 14:00 fixup.dat
# -rwxrwxrwx  1 ciro.cavani  staff       5.3K Mar 15 14:00 fixup4.dat
# -rwxrwxrwx  1 ciro.cavani  staff       3.1K Mar 15 14:00 fixup4cd.dat
# -rwxrwxrwx  1 ciro.cavani  staff       8.2K Mar 15 14:00 fixup4db.dat
# -rwxrwxrwx  1 ciro.cavani  staff       8.2K Mar 15 14:00 fixup4x.dat
# -rwxrwxrwx  1 ciro.cavani  staff       3.1K Mar 15 14:00 fixup_cd.dat
# -rwxrwxrwx  1 ciro.cavani  staff        10K Mar 15 14:00 fixup_db.dat
# -rwxrwxrwx  1 ciro.cavani  staff        10K Mar 15 14:00 fixup_x.dat
# -rwxrwxrwx  1 ciro.cavani  staff        11M Mar 15 14:08 initramfs8
# -rwxrwxrwx  1 ciro.cavani  staff        11M Mar 15 14:08 initramfs_2712
# -rwxrwxrwx  1 ciro.cavani  staff       145B Mar 15 14:08 issue.txt
# -rwxrwxrwx  1 ciro.cavani  staff       8.8M Mar 15 14:00 kernel8.img
# -rwxrwxrwx  1 ciro.cavani  staff       8.8M Mar 15 14:00 kernel_2712.img
# drwxrwxrwx  1 ciro.cavani  staff        28K Mar 15 14:08 overlays
# -rwxrwxrwx  1 ciro.cavani  staff       2.8M Mar 15 14:00 start.elf
# -rwxrwxrwx  1 ciro.cavani  staff       2.2M Mar 15 14:00 start4.elf
# -rwxrwxrwx  1 ciro.cavani  staff       790K Mar 15 14:00 start4cd.elf
# -rwxrwxrwx  1 ciro.cavani  staff       3.6M Mar 15 14:00 start4db.elf
# -rwxrwxrwx  1 ciro.cavani  staff       2.9M Mar 15 14:00 start4x.elf
# -rwxrwxrwx  1 ciro.cavani  staff       790K Mar 15 14:00 start_cd.elf
# -rwxrwxrwx  1 ciro.cavani  staff       4.6M Mar 15 14:00 start_db.elf
# -rwxrwxrwx  1 ciro.cavani  staff       3.6M Mar 15 14:00 start_x.elf

cp ./boot/bcm2710-rpi-3-b-plus.dtb .
cp ./boot/overlays/disable-bt.dtbo .
cp ./boot/kernel8.img .

# Setting pi user password
# first "pi" (echo) -> password
# second "pi" (xargs) -> user

echo "pi" | openssl passwd -6 -stdin | xargs -0 printf 'pi:%s' > ./boot/userconf

# Activate SSH

touch ./boot/ssh


umount ./boot
hdiutil eject /dev/disk4
# "disk4" ejected.


brew install qemu

qemu-img --version
# qemu-img version 8.2.1
# Copyright (c) 2003-2023 Fabrice Bellard and the QEMU Project developers

qemu-system-aarch64 --version
# QEMU emulator version 8.2.1
# Copyright (c) 2003-2023 Fabrice Bellard and the QEMU Project developers

qemu-img resize -f raw 2024-03-15-raspios-bookworm-arm64-lite.img 8G
# Image resized.

# Compile a customized DTB

docker run --rm -it -v `pwd`:/rpi:rw debian:bookworm-slim

# (Docker container initialized)

export DEBIAN_FRONTEND=noninteractive
apt update
apt full-upgrade -y
apt install -y --no-install-recommends \
build-essential \
ca-certificates \
cmake \
git \
libfdt-dev

cd /rpi
git clone https://github.com/raspberrypi/utils.git rpi-utils

cd rpi-utils/dtmerge
cmake .
make
make install
which dtmerge
# /usr/local/bin/dtmerge
cd ../..

# (dtparam=uart0=on)
dtmerge \
bcm2710-rpi-3-b-plus.dtb \
bcm2710-rpi-3-b-plus_uart0-on.dtb \
- uart0=on

# (dtoverlay=disable-bt)
dtmerge \
bcm2710-rpi-3-b-plus_uart0-on.dtb \
bcm2710-rpi-3-b-plus_uart0-on_bt-off.dtb \
disable-bt.dtbo

exit

# (Docker container terminated)

echo '#!/usr/bin/env bash
set -eu

cd $(dirname "$0")

qemu-system-aarch64 \
-machine raspi3b \
-cpu cortex-a72 \
-nographic \
-dtb bcm2710-rpi-3-b-plus_uart0-on_bt-off.dtb \
-m 1G \
-smp 4 \
-kernel kernel8.img \
-drive if=sd,index=0,file=2024-03-15-raspios-bookworm-arm64-lite.img,format=raw \
-append "root=/dev/mmcblk0p2 rootdelay=1" \
-device usb-net,netdev=net0 \
-netdev user,id=net0,hostfwd=tcp::2222-:22' \
> start-rpi-qemu.sh

chmod +x start-rpi-qemu.sh

./start-rpi-qemu.sh
# (...)
#
# Debian GNU/Linux 12 raspberrypi ttyAMA0
#
# raspberrypi login:
# (terminal blocked, show system messages like kernel panic and such)

# Terminal 2

echo '#!/usr/bin/env bash
set -eu

ssh -o StrictHostKeyChecking=accept-new -p 2222 pi@localhost' \
> rpi-shell.sh

chmod +x rpi-shell.sh

./rpi-shell.sh
# pi@localhost's password: pi

# (Raspberry Pi shell initialized)

sudo apt update
sudo apt full-upgrade -y --no-install-recommends
sudo apt clean

df -h
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/root       2.0G  1.7G  184M  91% /
# devtmpfs        426M     0  426M   0% /dev
# tmpfs           460M     0  460M   0% /dev/shm
# tmpfs           184M  764K  184M   1% /run
# tmpfs           5.0M     0  5.0M   0% /run/lock
# /dev/mmcblk0p1  510M   63M  448M  13% /boot/firmware
# tmpfs            92M     0   92M   0% /run/user/1000


# resize root partition

sudo raspi-config
# 6 Advanced Options - Configure advanced settings
# A1 Expand Filesystem - Ensure that all of the SD card is available
# > The filesystem will be enlarged upon the next reboot
# ESC ESC

sudo halt

# (Raspberry Pi shell terminated)

# (Terminal 1 - QEMU)
# start RPI again

./start-rpi-qemu.sh

# Terminal 2

./rpi-shell.sh
# pi@localhost's password: pi

# (Raspberry Pi shell initialized)

df -h
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/root       7.4G  1.8G  5.3G  25% /
# devtmpfs        426M     0  426M   0% /dev
# tmpfs           460M     0  460M   0% /dev/shm
# tmpfs           184M  764K  184M   1% /run
# tmpfs           5.0M     0  5.0M   0% /run/lock
# /dev/mmcblk0p1  510M   63M  448M  13% /boot/firmware
# tmpfs            92M     0   92M   0% /run/user/1000

# (`/dev/root` 2.0G -> 7.4G)

sudo halt

# (Raspberry Pi shell terminated)


# (Optional - cleanup)

rm -rf 2024-03-15-raspios-bookworm-arm64-lite.img.xz \
2024-03-15-raspios-bookworm-arm64-lite.img.xz.sha256 \
boot/ \
rpi-utils/ \
bcm2710-rpi-3-b-plus.dtb \
bcm2710-rpi-3-b-plus_uart0-on.dtb \
disable-bt.dtbo

ls -alh
# -rw-r--r--   1 ciro.cavani  709999406   8.0G Apr 22 10:54 2024-03-15-raspios-bookworm-arm64-lite.img
# -rw-r--r--   1 ciro.cavani  709999406    33K Apr 21 22:16 bcm2710-rpi-3-b-plus_uart0-on_bt-off.dtb
# -rwxr-xr-x   1 ciro.cavani  709999406   8.8M Apr 21 22:13 kernel8.img
# -rwxr-xr-x   1 ciro.cavani  709999406    90B Apr 22 11:00 rpi-shell.sh
# -rw-r--r--   1 ciro.cavani  709999406   407B Apr 22 10:47 start-rpi-qemu.sh
```
