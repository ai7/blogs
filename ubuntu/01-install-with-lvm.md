# Install Ubuntu With LVM

Install Ubuntu and setup the disk using Logical Volume Management (LVM).

<!-- TOC depthFrom:2 -->

- [1. Introduction](#1-introduction)
- [2. Overview](#2-overview)
- [3. Install Ubuntu](#3-install-ubuntu)
    - [3.1. Start Installer](#31-start-installer)
    - [3.2. Install Wizard](#32-install-wizard)
        - [3.2.1. Enable LVM Option](#321-enable-lvm-option)
        - [3.2.2. Continue Wizard](#322-continue-wizard)
    - [3.3. Install Start](#33-install-start)
    - [3.4. Reboot to Desktop](#34-reboot-to-desktop)
- [4. System Details](#4-system-details)
    - [4.1. lsblk](#41-lsblk)
    - [4.2. df](#42-df)
    - [4.3. fstab](#43-fstab)
    - [4.4. lvm](#44-lvm)
- [5. Conclusion](#5-conclusion)

<!-- /TOC -->

## 1. Introduction

It is good practice to put Operating System (OS) and data files on
separate partitions. This separation allows you to manage the OS and
data files more easily and efficiently. This is applicable regardless
of the Operating System in use.

For example, you can cleanly re-install the OS (wiping the OS
partition) without affecting your data files. You can easily back up
all your data files without the trouble of figuring out where the data
files are, or incur the overhead of backing up unnecessary operating
system files.

Unfortunately, not all systems allow you to easily configure this
OS/data separation at install time, including Ubuntu. This typically
needs to be done manually after the OS is installed. It can be
relatively straightforward if the system have multiple hard drives.
Simply install the OS on one, and use the others for data. But it is
less trivial if the system only have one hard drive, as is the case
for many systems and laptops. Partitioning is necessary to achieve
this OS/data separation, or to better optimize OS/data storage
allocations.

## 2. Overview

[LVM](https://wiki.ubuntu.com/Lvm) stands for Logical Volume
Management. It is more advanced and flexible than traditional disk
partitioning.

This guide shows you how to install Ubuntu using LVM so you can manage
your partitions with greater ease and flexibility. For example, you
can easily resize the root partition, and create a separate Data
partition to hold all your data files on the same hard disk.

Note: The installation and screenshots below are from **Ubuntu 20.04
LTS (Focal Fossa)**. The installation process may be slightly
different on other Ubuntu versions, but the basic idea remains the
same.

## 3. Install Ubuntu

The Ubuntu installer supports LVM directly. All that is needed is to
enable it when formatting your hard drive during the install process.

The installer however doesn't provide an easy way create a separate
partition for your data files (`/home`) during the installation
process. This can be done after the install.

### 3.1. Start Installer

Boot from the Ubuntu install CD or USB. Press `Space Bar` when you see
the keyboard logo below:

![Imgur](https://i.imgur.com/8HBXNd8.png)

You will be presented with the language selection screen:

![Imgur](https://i.imgur.com/pHWIoDt.png)

Select the language of your choice.

![Imgur](https://i.imgur.com/gl1KhdX.png)

Select the `Install Ubuntu` option to start the installer.

### 3.2. Install Wizard

Continue with the install wizard until you reach the disk page.

![Imgur](https://i.imgur.com/ISxxxZa.png)

![Imgur](https://i.imgur.com/1v2kQAn.png)

![Imgur](https://i.imgur.com/DYXxN23.png)

#### 3.2.1. Enable LVM Option

This is where you enable **LVM** for the installation.

- In the **Installation type** screen, select the **Erase disk** option.
- Click the **Advanced features...** button, and select the **Use LVM** option.

![Imgur](https://i.imgur.com/JOmagxu.png)

Click `Install Now`. You will see a confirmation screen shortly:

![Imgur](https://i.imgur.com/lGixieQ.png)

This screen shows you the following information:

- Hard disk `sda` will be used.
- LVM group `vgubuntu` will be created.
- LVM volume `root` and `swap_1` will be created inside the LVM `vgubuntu` group.

#### 3.2.2. Continue Wizard

Continue with the installation wizard:

![Imgur](https://i.imgur.com/UqNXDsv.png)

This will be the last screen before the installation starts:

![Imgur](https://i.imgur.com/1Qo8UDz.png)

Press `Continue` to start start the installations.

### 3.3. Install Start

The installation and file copying will start at this point:

![Imgur](https://i.imgur.com/3Xaucqz.png)

![Imgur](https://i.imgur.com/uZuPoKU.png)

After installation is completed, you'll be prompted for reboot:

![Imgur](https://i.imgur.com/jQzhL1U.png)

### 3.4. Reboot to Desktop

Remove the installation medium, and reboot into the new OS:

![Imgur](https://i.imgur.com/rpdRQfS.png)

If all goes well, you should see the Login Screen:

![Imgur](https://i.imgur.com/qxoKU5P.png)

After login, some minor configuration/info page:

![Imgur](https://i.imgur.com/QFmwwfo.png)

The Ubuntu 20.04 desktop after all is done:

![Imgur](https://i.imgur.com/AvgduPc.png)

## 4. System Details

Let's take a look at the system to see how the disk is setup:

### 4.1. lsblk

```console
vmuser@ubt-202006:~$ lsblk -i
NAME                MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
loop0                 7:0    0    55M  1 loop /snap/core18/1754
loop1                 7:1    0    55M  1 loop /snap/core18/1705
loop2                 7:2    0 240.8M  1 loop /snap/gnome-3-34-1804/24
loop3                 7:3    0  62.1M  1 loop /snap/gtk-common-themes/1506
loop4                 7:4    0  49.8M  1 loop /snap/snap-store/433
loop5                 7:5    0  27.1M  1 loop /snap/snapd/7264
loop6                 7:6    0  29.8M  1 loop /snap/snapd/8140
loop7                 7:7    0  49.8M  1 loop /snap/snap-store/467
sda                   8:0    0    80G  0 disk
|-sda1                8:1    0   512M  0 part /boot/efi
|-sda2                8:2    0     1K  0 part
`-sda5                8:5    0  79.5G  0 part
  |-vgubuntu-root   253:0    0  78.6G  0 lvm  /
  `-vgubuntu-swap_1 253:1    0   980M  0 lvm  [SWAP]
sr0                  11:0    1  1024M  0 rom
```

The 80G hard drive `sda` is partitioned into the following 3 physical partitions:
- `sda1`: 512M partition for `/boot/efi`
- `sda2`: 1K partition (for ?)
- `sda5`: remaining 79.5G partition used by LVM, with the following 2 logical partitions:
  - `vgubuntu-swap_1`: 980M logical SWAP partition
  - `vgubuntu-root`: remaining 78.6G logical root partition for `/`.

### 4.2. df

Here's a different view of the various mount points and file systems:

```console
vmuser@ubt-202006:~$ df -Th
Filesystem                Type      Size  Used Avail Use% Mounted on
udev                      devtmpfs  3.9G     0  3.9G   0% /dev
tmpfs                     tmpfs     794M  1.8M  792M   1% /run
/dev/mapper/vgubuntu-root ext4       77G  5.8G   68G   8% /
tmpfs                     tmpfs     3.9G     0  3.9G   0% /dev/shm
tmpfs                     tmpfs     5.0M  4.0K  5.0M   1% /run/lock
tmpfs                     tmpfs     3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/loop0                squashfs   55M   55M     0 100% /snap/core18/1754
/dev/loop2                squashfs  241M  241M     0 100% /snap/gnome-3-34-1804/24
/dev/loop3                squashfs   63M   63M     0 100% /snap/gtk-common-themes/1506
/dev/loop4                squashfs   50M   50M     0 100% /snap/snap-store/433
/dev/loop1                squashfs   55M   55M     0 100% /snap/core18/1705
/dev/loop6                squashfs   30M   30M     0 100% /snap/snapd/8140
/dev/loop5                squashfs   28M   28M     0 100% /snap/snapd/7264
/dev/sda1                 vfat      511M  4.0K  511M   1% /boot/efi
/dev/loop7                squashfs   50M   50M     0 100% /snap/snap-store/467
tmpfs                     tmpfs     794M   24K  794M   1% /run/user/1000
/dev/loop8                squashfs  256M  256M     0 100% /snap/gnome-3-34-1804/36
```

`squashfs` is a compressed read-only file system for Linux. `Snap` is
a software deployment and package management system for Linux.

As you can see, various core Ubuntu subsystems are bundled into
self-contained snaps, and packaged as read-only squashfs. They are
mounted as read-only directories in specific target locations. This
improves both security and upgradibility. Many modern systems
([ESX](https://en.wikipedia.org/wiki/VMware_ESXi),
[MacOS](https://en.wikipedia.org/wiki/MacOS), etc) applies similar
principles to manage its core OS files for improved stability and
resiliency.

### 4.3. fstab

The system have 3 mount points defined in `/etc/fstab`:

```
vmuser@ubt-202006:~$ cat /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
/dev/mapper/vgubuntu-root /               ext4    errors=remount-ro 0       1
# /boot/efi was on /dev/sda1 during installation
UUID=B92F-3466  /boot/efi       vfat    umask=0077      0       1
/dev/mapper/vgubuntu-swap_1 none            swap    sw              0       0
```

### 4.4. lvm

Here are the LVM information with `pvs` and `lvs` commands:

```console
vmuser@ubt-202006:~$ sudo pvs
  PV         VG       Fmt  Attr PSize   PFree
  /dev/sda5  vgubuntu lvm2 a--  <79.50g    0
```

```console
vmuser@ubt-202006:~$ sudo lvs
  LV     VG       Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root   vgubuntu -wi-ao---- <78.54g
  swap_1 vgubuntu -wi-ao---- 980.00m
```

More detailed information are available with the `pvdisplay` and `lvdisplay` commands.

## 5. Conclusion

Now that we've setup the system with LVM, we can easily resize the
root partition to reduce its size, and create a new data partition in
the `vgubuntu` group using the freed up space. We can then relocate
the entire `/home` directory to this new partition.

See the next guide [create home partition](02-create-home-with-lvm.md) for more information.
