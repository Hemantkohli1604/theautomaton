title: Extract partition data from Xen image file
author: Hemant Kohli
date: 2019-09-18 00:46:12
tags:
---
A while ago I faced the famous "ERROR: boot loader didn't return any data!" message while booting a Xen machine. The error suggests that there is a problem with booting the VM. 

After performing the initial troubleshooting I was facing issues in recovering the VM. The best solution was to anyhow get the data from the .img file for the VM.

In comes the Kpartx tool that can help you with the same. 

losetup -f

``/dev/loop0``


This shows the available physical loop file/device that is unused and can be used to add a loop device.

losetup /dev/loop0 /HDD_500/images/javapostgresql.img

Associated the image file to loop file/device

kpartx -av /dev/loop0

```add map loop0p1 : 0 1024000 linear /dev/loop0 2048 add map loop0p2 : 0 60413952 linear /dev/loop0 1026048```

This command reads partition tables on specified device and create device
maps over partitions segments detected

losetup -a `[lists all used loop files ]`

```/dev/loop0: [0811]:32571394 (/HDD_500GB/images/javapostgresql.img)```

Next run vgscan

vgscan

```
Reading all physical volumes.  This may take a while...
Found volume group "VolGroup" using metadata type lvm2
```

vgchange -ay VolGroup
```
2 logical volume(s) in volume group "VolGroup" now active```

activate and deactivate VolumeGroupName, or all volume groups if none is specified.

lvs
```
LV      VG       Attr   LSize  Origin Snap%  Move Log Copy%  Convert
lv_root VolGroup -wi-a- 27.80G
lv_swap VolGroup -wi-a-  1.00G```


mount /dev/VolGroup/lv_root /mnt/data/

Now you can access your root partition in image on /mnt/data folder

After data is copied unmount the partition
umount /mnt/data/

vgchange -an VolGroup
```
0 logical volume(s) in volume group "VolGroup" now active```

kpartx -d /dev/loop0
losetup -d /dev/loop0 ```[delete from used]```

NOTE: always remember to deactivate the logical volumes with "vgchange -an", remove the partitions with "kpartx -d", and (if appropriate) delete the loop device with "losetup -d" after you are finished. There are two reasons: first of all, the default volume group name for a install is always the same, so if you end up activating two disk images at the same time you'll end up with two separate LVM volume groups with the same name. LVM will cope as best it can, but you won't be able to distinguish between these two groups on the command line.

And secondly, if you don't deactivate it, then if the guest is started up again, you might end up with the LVM being active in both the guest and the dom0 at the same time, and this may lead to VG or filesystem corruption.

