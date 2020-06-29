# Install Ubuntu With LVM

## Overview

[LVM](https://wiki.ubuntu.com/Lvm) stands for Logical Volume Management. It is much
more advanced and flexible than traditional disk partitioning.

This guide shows you how to install Ubuntu using LVM so you can manage your partitions
with greater ease and flexibility.

## Start Ubuntu Installer

The Ubuntu installer supports LVM directly. All you need to do is to enable it
when formatting your hard drive during the install process.

Boot from the Ubuntu install CD or USB. Press `Space Bar` when you see the keyboard logo below:

![Imgur](https://i.imgur.com/8HBXNd8.png)

You should see the language selection screen:

![Imgur](https://i.imgur.com/pHWIoDt.png)

Select the language of your choice. You will then see the install options:

![Imgur](https://i.imgur.com/gl1KhdX.png)

- Select the `Install Ubuntu` option to start the install process.

## Enable LVM Option

When the installer gets to the disk page, select the "Erase disk" option,
then click "Advanced features..." button and enable "Use LVM".

![Imgur](https://i.imgur.com/JOmagxu.png)

- Click "Install Now", you will see the confirmation dialog shortly:

![Imgur](https://i.imgur.com/lGixieQ.png)

This screen shows you that Ubuntu will create a LVM group `vgubuntu` on hard drive `sda`,
and create a `root` and `swap_1` volume inside the LVM `vgubuntu` group.

## Finish the Install

Continue and finish the install. Reboot the computer when required.
