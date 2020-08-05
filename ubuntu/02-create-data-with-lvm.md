---
layout: post
title:  "Create Data Partition With LVM"
---

# Create Data Partition With LVM

Create separate data partition using Logical Volume Management (LVM).

<!-- TOC depthFrom:2 -->

- [1. Introduction](#1-introduction)
- [2. Overview](#2-overview)
- [3. Create Data Partition](#3-create-data-partition)
    - [3.1. Resize Root Partition](#31-resize-root-partition)
        - [3.1.1. Boot Live CD](#311-boot-live-cd)
        - [3.1.2. Current Disk Status](#312-current-disk-status)
        - [3.1.3. Resize Root](#313-resize-root)
    - [3.2. Create Data Partition](#32-create-data-partition)
        - [3.2.1. Create Partition](#321-create-partition)
        - [3.2.2. Format Partition](#322-format-partition)
        - [3.2.3. Disk Status](#323-disk-status)
    - [3.3. Copy Data](#33-copy-data)
    - [3.4. Mount Partition](#34-mount-partition)
- [4. System Details](#4-system-details)
    - [4.1. lsblk](#41-lsblk)
    - [4.2. df](#42-df)
    - [4.3. fstab](#43-fstab)
    - [4.4. lvm](#44-lvm)
    - [4.5. Disks](#45-disks)
    - [4.6. Disk Usage Analyzer](#46-disk-usage-analyzer)
- [5. Conclusion](#5-conclusion)

<!-- /TOC -->

## 1. Introduction

In the previous guide, we installed Ubuntu and setup the disk with
LVM. We are now ready to create our data partition. Having a separate
data partition allows us to put the entire `/home` directory there,
and enjoy all ease and benefits of having our data files reside on a
dedicated partition.

## 2. Overview

The existing `root` partition uses all the available space on the hard
drive. This guide shows you how to reduce the root partition size, and
create a new data partition from the remaining space. We'll then copy
over the contents of exisitng `/home` to the new partition, and
finally mount the new partition under `/home`.

The main commands used are:

```bash
# resize root partition
$ sudo lvreduce --resizefs --size 30g /dev/vgubuntu/root
```
```bash
# create data partition
$ sudo lvcreate -n mydata -l 100%FREE vgubuntu
```
```bash
# format data partition
$ sudo mkfs.ext4 /dev/vgubuntu/mydata
```
```bash
# mirror existing /home to new data partition
$ sudo rsync -aXSP --stats /home/. /home2/.
```

The non trivial part is we need to do some of these steps with Live CD, as we
can't resize the root partition while the OS is booted from it. Details below.

## 3. Create Data Partition

### 3.1. Resize Root Partition

To resize the root partition, first decide how big you want it to be.
This will store the OS, plus any programs you will be installing, but
not your data files.

Based on the total disk size, decide how much you want to allocate for
the OS and data partition respectively based on your use case. (From
the previous guide, we saw Ubuntu 20.04 itself took about 5.8G after a
normal installation.)

As an example, if you have a 128G SSD, you may decide you want 32G for
OS, and the remaining 96G for data. Or split 50/50 and do 64G/64G.
Generally, the bigger the hard drive, the less percentage the OS
partition would normally take, since the OS size is generally fixed
and well managed, where as your data could be quite big depending on
what you are storing.

#### 3.1.1. Boot Live CD

![Imgur](https://i.imgur.com/UfJa6Vs.png)

![Imgur](https://i.imgur.com/S5SMffF.png)

![Imgur](https://i.imgur.com/WGhOKMc.png)

#### 3.1.2. Current Disk Status

Let's check the disk status when booted from live CD:

```console
ubuntu@ubuntu:~$ lsblk -i
NAME                MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
loop0                 7:0    0   1.9G  1 loop /rofs
loop1                 7:1    0  27.1M  1 loop /snap/snapd/7264
loop2                 7:2    0    55M  1 loop /snap/core18/1705
loop3                 7:3    0 240.8M  1 loop /snap/gnome-3-34-1804/24
loop4                 7:4    0  62.1M  1 loop /snap/gtk-common-themes/1506
loop5                 7:5    0  49.8M  1 loop /snap/snap-store/433
sda                   8:0    0    80G  0 disk 
|-sda1                8:1    0   512M  0 part 
|-sda2                8:2    0     1K  0 part 
`-sda5                8:5    0  79.5G  0 part 
  |-vgubuntu-root   253:0    0  78.6G  0 lvm  
  `-vgubuntu-swap_1 253:1    0   980M  0 lvm  
sr0                  11:0    1   2.5G  0 rom  /cdrom
```

Notice lvm volume `vgubuntu-root` and `vgubuntu-swap_1` are correctly identified,
and not mounted to anything.

```
ubuntu@ubuntu:~$ df -Th
Filesystem     Type      Size  Used Avail Use% Mounted on
udev           devtmpfs  3.9G     0  3.9G   0% /dev
tmpfs          tmpfs     794M  1.8M  792M   1% /run
/dev/sr0       iso9660   2.6G  2.6G     0 100% /cdrom
/dev/loop0     squashfs  2.0G  2.0G     0 100% /rofs
/cow           overlay   3.9G  481M  3.5G  13% /
tmpfs          tmpfs     3.9G     0  3.9G   0% /dev/shm
tmpfs          tmpfs     5.0M  4.0K  5.0M   1% /run/lock
tmpfs          tmpfs     3.9G     0  3.9G   0% /sys/fs/cgroup
tmpfs          tmpfs     3.9G     0  3.9G   0% /tmp
tmpfs          tmpfs     794M   60K  794M   1% /run/user/999
/dev/loop1     squashfs   28M   28M     0 100% /snap/snapd/7264
/dev/loop2     squashfs   55M   55M     0 100% /snap/core18/1705
/dev/loop3     squashfs  241M  241M     0 100% /snap/gnome-3-34-1804/24
/dev/loop4     squashfs   63M   63M     0 100% /snap/gtk-common-themes/1506
/dev/loop5     squashfs   50M   50M     0 100% /snap/snap-store/433
```

Notice the root directory `/` is **not** from `/dev/mapper/vgubuntu-root` and not `ext4`, as will be the case if
the system is booted normally.

Verify `pvs` and `lvs` shows expected information.

```console
ubuntu@ubuntu:~$ sudo pvs
  PV         VG       Fmt  Attr PSize   PFree
  /dev/sda5  vgubuntu lvm2 a--  <79.50g    0 
```
```console
ubuntu@ubuntu:~$ sudo lvs
  LV     VG       Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root   vgubuntu -wi-a----- <78.54g                                                    
  swap_1 vgubuntu -wi-a----- 980.00m                                                    
```

#### 3.1.3. Resize Root

The command to resize the lvm root partition is:

```bash
sudo lvreduce --resizefs --size 30g /dev/vgubuntu/root
```
- substitute `30g` with the size you want for your OS partition.
- substitute `vgubuntu` and `root` with the actual lvm **group** and **volume** name.

Here's what it looks like when the command is run:

```console
ubuntu@ubuntu:~$ sudo lvreduce --resizefs --size 30g /dev/vgubuntu/root
fsck from util-linux 2.34
/dev/mapper/vgubuntu-root: 192359/5152768 files (0.2% non-contiguous), 1963548/20588544 blocks
resize2fs 1.45.5 (07-Jan-2020)
Resizing the filesystem on /dev/mapper/vgubuntu-root to 7864320 (4k) blocks.
The filesystem on /dev/mapper/vgubuntu-root is now 7864320 (4k) blocks long.

  Size of logical volume vgubuntu/root changed from <78.54 GiB (20106 extents) to 30.00 GiB (7680 extents).
  Logical volume vgubuntu/root successfully resized.
```

Run `pvs` and `lvs` to see the updated LVM status:

```console
ubuntu@ubuntu:~$ sudo pvs
  PV         VG       Fmt  Attr PSize   PFree  
  /dev/sda5  vgubuntu lvm2 a--  <79.50g <48.54g
```

`PFree` now shows the freed up space.

```console
ubuntu@ubuntu:~$ sudo lvs
  LV     VG       Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root   vgubuntu -wi-a-----  30.00g                                                    
  swap_1 vgubuntu -wi-a----- 980.00m                                                    
```

We see the updated size for root, `30.00g` in our case.

### 3.2. Create Data Partition

We are now ready to create the data partition on the free space in
`vgubuntu` group.

#### 3.2.1. Create Partition

The command to create a lvm partition is below, depending on what you
want the size to be:

```bash
# create partition with all remaining free space
sudo lvcreate -n mydata -l 100%FREE vgubuntu
```
```bash
# create partition with fixed size
sudo lvcreate -n mydata -L 10g vgubuntu
```

Here's what it looks like when the command is run:

```console
ubuntu@ubuntu:~$ sudo lvcreate -n mydata -l 100%FREE vgubuntu
  Logical volume "mydata" created.
```

#### 3.2.2. Format Partition

The command to format the newly created partition is:

```bash
$ sudo mkfs.ext4 /dev/vgubuntu/mydata
```

Here's what it looks like when the command is run:

```console  
ubuntu@ubuntu:~$ sudo mkfs.ext4 /dev/vgubuntu/mydata 
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 12724224 4k blocks and 3186688 inodes
Filesystem UUID: a23ccedf-eeda-488c-859e-66f4d65dc00a
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000, 7962624, 11239424

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (65536 blocks): done
Writing superblocks and filesystem accounting information: done   
```

At this point, the data partition has been created, and formatted with
the `ext4` file system.

#### 3.2.3. Disk Status

Let's check the updated lvm disk status:

```console
ubuntu@ubuntu:~$ sudo pvs
  PV         VG       Fmt  Attr PSize   PFree
  /dev/sda5  vgubuntu lvm2 a--  <79.50g    0 
```

We see `PFree` is back to 0.

```console
ubuntu@ubuntu:~$ sudo lvs
  LV     VG       Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  mydata vgubuntu -wi-a----- <48.54g                                                    
  root   vgubuntu -wi-a-----  30.00g                                                    
  swap_1 vgubuntu -wi-a----- 980.00m                                                    
```

We see the new lvm volume `mydata` and its size.

```console
ubuntu@ubuntu:~$ lsblk -i
NAME                MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
loop0                 7:0    0   1.9G  1 loop /rofs
loop1                 7:1    0  27.1M  1 loop /snap/snapd/7264
loop2                 7:2    0    55M  1 loop /snap/core18/1705
loop3                 7:3    0 240.8M  1 loop /snap/gnome-3-34-1804/24
loop4                 7:4    0  62.1M  1 loop /snap/gtk-common-themes/1506
loop5                 7:5    0  49.8M  1 loop /snap/snap-store/433
sda                   8:0    0    80G  0 disk 
|-sda1                8:1    0   512M  0 part 
|-sda2                8:2    0     1K  0 part 
`-sda5                8:5    0  79.5G  0 part 
  |-vgubuntu-root   253:0    0    30G  0 lvm  
  |-vgubuntu-swap_1 253:1    0   980M  0 lvm  
  `-vgubuntu-mydata 253:2    0  48.6G  0 lvm  
sr0                  11:0    1   2.5G  0 rom  /cdrom
```

We see the new `vgubuntu-mydata` with **lsblk**, and it's not mounted,
as expected.

### 3.3. Copy Data

To set the new data partition as `/home`, we need to copy over th
existing content of `/home` to the new partition. To do this, we need
to have both the lvm partitions mounted so we can operate on its
contents.

```bash
# create temporary mount point
sudo mkdir /root1 /home2
sudo mount /dev/vgubuntu/root   /root1
sudo mount /dev/vgubuntu/mydata /home2
```

```bash
# copy data from /home to /home2
sudo rsync -aXSP --stats /root1/home/. /home2/.
```

```bash
# diff to check
sudo diff -r /root1/home /home2
```

### 3.4. Mount Partition

```bash
# update fstab
sudo nano /root1/etc/fstab
```

```
/dev/mapper/vgubuntu-home /home           ext4    defaults    0       2
```

```bash
# rename old home
cd /root1 && sudo mv home old_home && sudo mkdir home
```

```
sudo umount /root1 /home2
```

reboot into the new OS.

## 4. System Details

Let's take a closer look at the system to see how the disk changed:

### 4.1. lsblk

```console
vmuser@ubt-202006:~$ lsblk
NAME                MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
loop0                 7:0    0 240.8M  1 loop /snap/gnome-3-34-1804/24
loop1                 7:1    0    55M  1 loop /snap/core18/1754
loop2                 7:2    0    55M  1 loop /snap/core18/1705
loop3                 7:3    0 255.6M  1 loop /snap/gnome-3-34-1804/36
loop4                 7:4    0  29.8M  1 loop /snap/snapd/8140
loop5                 7:5    0  49.8M  1 loop /snap/snap-store/433
loop6                 7:6    0  49.8M  1 loop /snap/snap-store/467
loop7                 7:7    0  62.1M  1 loop /snap/gtk-common-themes/1506
loop8                 7:8    0  27.1M  1 loop /snap/snapd/7264
sda                   8:0    0    80G  0 disk 
├─sda1                8:1    0   512M  0 part /boot/efi
├─sda2                8:2    0     1K  0 part 
└─sda5                8:5    0  79.5G  0 part 
  ├─vgubuntu-root   253:0    0    30G  0 lvm  /
  ├─vgubuntu-swap_1 253:1    0   980M  0 lvm  [SWAP]
  └─vgubuntu-mydata 253:2    0  48.6G  0 lvm  /home
sr0                  11:0    1  1024M  0 rom  
```

- Notice the new `48.6G` volume `vgubuntu-mydata` mounted at `/home`.

### 4.2. df

Here's a different view of the various mount points and file systems:

```console
vmuser@ubt-202006:~$ df -Th
Filesystem                  Type      Size  Used Avail Use% Mounted on
udev                        devtmpfs  3.9G     0  3.9G   0% /dev
tmpfs                       tmpfs     794M  1.9M  792M   1% /run
/dev/mapper/vgubuntu-root   ext4       30G  5.8G   22G  21% /
tmpfs                       tmpfs     3.9G     0  3.9G   0% /dev/shm
tmpfs                       tmpfs     5.0M  4.0K  5.0M   1% /run/lock
tmpfs                       tmpfs     3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/loop0                  squashfs  241M  241M     0 100% /snap/gnome-3-34-1804/24
/dev/loop1                  squashfs   55M   55M     0 100% /snap/core18/1754
/dev/loop2                  squashfs   55M   55M     0 100% /snap/core18/1705
/dev/loop4                  squashfs   30M   30M     0 100% /snap/snapd/8140
/dev/loop3                  squashfs  256M  256M     0 100% /snap/gnome-3-34-1804/36
/dev/loop6                  squashfs   50M   50M     0 100% /snap/snap-store/467
/dev/loop8                  squashfs   28M   28M     0 100% /snap/snapd/7264
/dev/loop5                  squashfs   50M   50M     0 100% /snap/snap-store/433
/dev/loop7                  squashfs   63M   63M     0 100% /snap/gtk-common-themes/1506
/dev/mapper/vgubuntu-mydata ext4       48G   62M   46G   1% /home
/dev/sda1                   vfat      511M  4.0K  511M   1% /boot/efi
tmpfs                       tmpfs     794M   20K  794M   1% /run/user/1000
```

- Notice `/dev/mapper/vgubuntu-mydata` is mounted at `/home`.

### 4.3. fstab

The system now have 4 mount points defined in `/etc/fstab`:

```
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
  mydata vgubuntu -wi-ao---- <48.54g                                                    
  root   vgubuntu -wi-ao----  30.00g                                                    
  swap_1 vgubuntu -wi-ao---- 980.00m                                                    
```

Here are more detailed information with `pvdisplay` and `lvdisplay` commands:

```console
vmuser@ubt-202006:~$ sudo pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda5
  VG Name               vgubuntu
  PV Size               <79.50 GiB / not usable 0   
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              20351
  Free PE               0
  Allocated PE          20351
  PV UUID               wYaz40-dKqp-FiFf-iRWi-N2wR-yuRu-3IL3k5
```

```console
vmuser@ubt-202006:~$ sudo lvdisplay
  --- Logical volume ---
  LV Path                /dev/vgubuntu/root
  LV Name                root
  VG Name                vgubuntu
  LV UUID                maVbwo-NPfn-m6oc-reEv-f9N7-e2kf-vMuj3Y
  LV Write Access        read/write
  LV Creation host, time ubuntu, 2020-06-29 08:12:15 -0700
  LV Status              available
  # open                 1
  LV Size                30.00 GiB
  Current LE             7680
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
   
  --- Logical volume ---
  LV Path                /dev/vgubuntu/swap_1
  LV Name                swap_1
  VG Name                vgubuntu
  LV UUID                GV9MRf-LXxc-1ZLH-eqpt-49FM-zwit-oFYPK9
  LV Write Access        read/write
  LV Creation host, time ubuntu, 2020-06-29 08:12:16 -0700
  LV Status              available
  # open                 2
  LV Size                980.00 MiB
  Current LE             245
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:1
   
  --- Logical volume ---
  LV Path                /dev/vgubuntu/mydata
  LV Name                mydata
  VG Name                vgubuntu
  LV UUID                LCQeaw-ANKK-PrrE-nSFb-v0TP-UlUN-GpCakG
  LV Write Access        read/write
  LV Creation host, time ubuntu, 2020-06-30 10:19:01 -0700
  LV Status              available
  # open                 1
  LV Size                <48.54 GiB
  Current LE             12426
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:2
```

### 4.5. Disks

Here's what the **Disk** utility program shows:

![Imgur](https://i.imgur.com/hc2AQIo.png)

Our `86 GB` hard disk `/dev/sda` is physically partitioned into 3 partitions:
- `sda1`: 537 MB FAT32 bootable [primary partition](https://en.wikipedia.org/wiki/Disk_partitioning#Primary_partition).
- `sda2`: 85 GB [extended partition](https://en.wikipedia.org/wiki/Disk_partitioning#Extended_partition) container.
- `sda5`: 85 GB Linux LVM partition containing a lvm physical volume (PV).

This is standard PC partitioning since the beginning of time.

What's unique and fascinating about LVM is that, within one physical
partition (`sda5`), we can have multiple **logical** volumes (think of
it as virtual partitions). You can expand, shrink, append space from
additional hard drive, etc, to logical volumes, all without touching
the physical and fragile PC partition table at all. That's power!

### 4.6. Disk Usage Analyzer

Last but not least, the **Disk Usage Analyzer** tool gives a nice view
of what is taking up space on your hard disk:

![Imgur](https://i.imgur.com/FvcPII8.png)

## 5. Conclusion

There you have it, we've installed Ubuntu on a system with LVM, and put
`/home` on a dedicated LVM volume, separate from the `root` volume.

At some point in the future, you may have the desire to blow away your OS
partition and start over again, without go to the next guide for details
on how to do this while keeping your existing data volume.
