# Installing Arch Linux ARM (ALARM) on the SHARP Zaurus SL-C3x00 series

### Introduction

This guide covers the process of installing a base Arch Linux system on the SHARP Zaurus SL-C3x00 series of PDAs. If you're new to Arch Linux, be warned that it is not designed for Linux newbs like Ubuntu and Mint are. Arch has quite a steep learning curve but it is a great way to learn Linux, you have easy access to many up-to-date packages and it is often more lightweight and faster than many other popular Linux distros.

The following instructions require that you have access to a working Linux computer with internet access and a SD card/Compact Flash device, a free SD or Compact Flash card of at least 1GB in size that can be used to install Arch and that you are comfortable using the Linux command line. You are also required to install the kexecboot bootloader before you can boot or install Arch. CF or USB ethernet adapters provide the easiest and most reliable way to access the internet under Arch and install additional packages.

### Why do I need a Linux box to install Arch on my Zaurus?

There are a few reasons why it is recommended you use another Linux computer to install ALARM:

* You will get faster disk read speeds if you install ALARM to a ext4 partition but the D+B busybox recovery console cannot format nor mount ext4 partitions so you need to format your internal drive and install Arch by booting Arch (or another Linux distro) from a removable SD or CF card.

* The ALARM base tarball is compressed with BSD tar so you cannot uncompress it with busybox tar.

* Having ALARM installed on an external SD or CF card acts as both a system backup and a more modern recovery OS than the D+B busybox console.

Windows and OSX lack the required tools to create the Arch installation media required to address these points.

### Install kexecboot

kexecboot is a bootloader similar to GRUB. kexecboot is needed to boot ALARM and hence it is required you install it first. It is recommended you use the ALARM kexecboot if you want your first kernel to be autobooted after 5 seconds.

To install kexecboot, download the tarball from the same page as the [Zaurus ALARM kernel downloads](https://github.com/greguu/linux-4.2.3-c3x00/releases) then extract its contents into the root directory of a FAT formatted SD or CF card. Completely turn off your Zaurus and insert the card with the kexec install files. Plug in the charger whilst holding the OK button then turn your Zaurus on. You should then see a grey, Japanese menu with four options. Choose option number four (update) then choose the device that contains the kexec install files. Your Z should then reboot, install kexecboot and then reboot again - this time into kexecboot.

### Create the Arch install medium

You can use either a Compact Flash or an SD card to install Arch to as long as its 1GB or larger. In this example I'm using a 4GB CF card that was pre-formatted as a vfat (Windows) drive so the first step is to insert the card to the card reader of a Linux PC and repartition and reformat the card as an ext4 Linux partition.

Before you repartition and format your card, you need to make sure you know the correct device name for the device you wish to format. You can use the `lsblk` or `dmesg` commands to see what device name your card has been assigned. If you get the device name wrong you risk losing data so be very careful! In my case the card device was `/dev/sdb` so I ran the following commands, which may require you to install `parted` first depending on your distro:

```sh
# umount /dev/sdb1
# parted -s -a optimal /dev/sdb mklabel msdos -- mkpart primary ext4 1 -1
# mkfs.ext4 /dev/sdb1
```

Your card should now be partitioned and formatted correctly for Arch so download the latest [Zaurus C3x00 ALARM rootfs tarball](https://github.com/greguu/alarm-zaurus-c3x00/releases). Presuming that the ALARM rootfs is saved within the current directory and your target card is mounted on /dev/sdb1 you'd run something like the following to extract the rootfs:

```sh
# mount /dev/sdb1 /mnt/
# bsdtar xvf alarm-zaurus-c3x00-minimal-rootfs-october2015.tar.xz -C /mnt/
# cp alarm-zaurus-c3x00-minimal-rootfs-october2015.tar.xz /mnt/root/
# umount /mnt/; sync
```

If you have installed Arch onto a CF card instead of an SD card you also need to edit `/mnt/boot/boot.cfg` and change `APPEND=root=/dev/mmcblk0p1` to `APPEND=root=/dev/sdb1`.

### Boot Arch

Insert your Arch install media into your Zaurus - you should now be able to boot into Arch. When you are presented with the login prompt you can login with the username and password root.

### Partition and format the internal drive

If you have not already done so you will need to partition your internal drive using `fdisk` or `cfdisk` before formatting the root partition as ext4 and running `mkswap` on your swap partition, if you decide to create one. 

I would create a swap partition of at least 256MB if you are attempting to compile large programs on your Zaurus. Note that there is a bug in systemd that prevents swap partitions being auto-mounted on boot so you will need to enable swap manually using the `swapon` command until this gets fixed.

### Extract the rootfs

When your internal drive has been partitioned and formatted, you can extract the rootfs tarball from the root dir onto the internal drive using similar mount and bsdtar commands to those used before. 

### Update boot.cfg and fstab

After extracting the rootfs onto the internal drive, the last step is to adjust the APPEND statement in `/boot/boot.cfg` to `APPEND=root=/dev/sda1` (or whatever device it will be booting from) and adjust the device name for the rootfs partition in `/etc/fstab`. Be sure to edit the versions of those files stored on the internal drive and not the copies on the external drive that you booted from.
