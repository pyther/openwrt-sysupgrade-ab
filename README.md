# OpenWrt A/B Upgrades
## Overview
A/B upgrades allow for safer OpenWrt upgrades. If the upgrade does not go as
planned, you can easily boot into your previous image.

The A/B upgrade approach functions by having two root parititons. When
executing an upgrade the new image will be written to the non-active root
partition. The grub config is updated to allow booting into either image.

At boot, you'll be able to select between either image. The most recently
installed/upgraded image will be the default.
* OpenWrt SNAPSHOT r15199-5d2b577a53
* OpenWrt SNAPSHOT r15129-d346beb08c

### Partition Layout
* /dev/sda1 = /boot
* /dev/sda2 = rootfs_a
* /dev/sda3 = rootfs_b


### High Level Process
1. Detect non-active partition
2. Create sysupgrade backup
3. Format partition
4. Mount partition to /mnt/sysupgrade
5. Extract rootfs to /mnt/sysupgrade
6. Copy kernel to /mnt/sysupgrade/vmlinuz
7. Extract openwrt-backup to /mnt/sysupgrade
8. Backup grub.cfg
9. Generate new grub.cfg
10. Reboot 

## Setup

1. Install Packages: `opkg install fdisk coreutils-stat blkid e2fsprogs`
2. Create partiton for second rootfs `fdisk /dev/sda`
    ```
    root@OpenWrt:~# fdisk /dev/sda

    Command (m for help): p
    Disk /dev/sda: 8 GiB, 8589934592 bytes, 16777216 sectors
    Device     Boot Start    End Sectors  Size Id Type
    /dev/sda1  *      512  33279   32768   16M 83 Linux
    /dev/sda2       33792 558079  524288  256M 83 Linux

    Command (m for help): n
    Select (default p): p
    Partition number (3,4, default 3): 3
    First sector (33280-16777215, default 559104):
    Last sector, +/-sectors or +/-size{K,M,G,T,P} (559104-16777215, default 16777215): +256M

    Created a new partition 3 of type 'Linux' and of size 256 MiB.

    Command (m for help): w
    The partition table has been altered.
    Syncing disks.
    ```
3. Copy current kernel to rootfs: `cp /boot/vmlinuz /vmlinuz`

## Upgrading
1. Modify upgrade.sh
    * Update `BOOT_DEV`, `ROOTA_DEV`, `ROOTB_DEV`
    * Update default grub options 
2. Install Packages: `opkg install coreutils-stat blkid`
3. Download rootfs and kernel to /root
     ```
     wget https://downloads.openwrt.org/snapshots/targets/x86/64/openwrt-x86-64-generic-kernel.bin
     wget https://downloads.openwrt.org/snapshots/targets/x86/64/openwrt-x86-64-rootfs.tar.gz
     ```
3. Upgrade: `./upgrade.sh openwrt-x86-64-generic-kernel.bin openwrt-x86-64-rootfs.tar.gz`
4. Reboot

