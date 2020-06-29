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

Unfortunately, not all systems allows you to easily configure this
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

## 3. Install Ubuntu

The Ubuntu installer supports LVM directly. All you need to do is to
enable it when formatting your hard drive during the install process.

The installer however doesn't provide an easy way create a separate
partition for your data files (/home) during the installation. This
can be done after the install.

### 3.1. Start Installer

Boot from the Ubuntu install CD or USB. Press `Space Bar` when you see
the keyboard logo below:

![Imgur](https://i.imgur.com/8HBXNd8.png)

You will be presented with the language selection screen:

![Imgur](https://i.imgur.com/pHWIoDt.png)

Select the language of your choice. You will then see the install options:

![Imgur](https://i.imgur.com/gl1KhdX.png)

Select the `Install Ubuntu` option to start the install process.

### 3.2. Install Wizard

Continue with the install wizard until you reach the disk selection
page.

![Imgur](https://i.imgur.com/ISxxxZa.png)

![Imgur](https://i.imgur.com/1v2kQAn.png)

![Imgur](https://i.imgur.com/DYXxN23.png)

#### 3.2.1. Enable LVM Option

This is the part where you enable LVM for the installation.

- In the **Installation type** screen, select the **Erase disk** option.
- Click the **Advanced features...** button, and select the **Use LVM** option.

![Imgur](https://i.imgur.com/JOmagxu.png)

Click "Install Now". You will see a confirmation dialog shortly:

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

Press `Continue` to start start the installations process.

### 3.3. Install Start

The installation and file copying will start at this point:

![Imgur](https://i.imgur.com/3Xaucqz.png)

![Imgur](https://i.imgur.com/uZuPoKU.png)

After installation is done, you'll be prompted for reboot:

![Imgur](https://i.imgur.com/jQzhL1U.png)

### 3.4. Reboot to Desktop

Remove the install media, and reboot into the new Desktop:

![Imgur](https://i.imgur.com/rpdRQfS.png)

If all goes well, you should see the Login Screen:

![Imgur](https://i.imgur.com/qxoKU5P.png)

Some minor configuration/info after login:

![Imgur](https://i.imgur.com/QFmwwfo.png)

The Ubuntu desktop after all is done:

![Imgur](https://i.imgur.com/AvgduPc.png)

## 4. System Details

Let's take a look at the system to see how LVM setup the disk.
