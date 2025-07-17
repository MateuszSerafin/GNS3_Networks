# Chapter 11: Logical Volume Management
LVM is pretty cool, it splits disks into volume groups, which are abstract, and you can partition modify them as you want without having an impact. Basically it abstracts the concept of partitions <br>

The structure of LVM consists of: 
- One or more disks configured as physical volumes (PV's)
- A volume group (VG) which is created using one or more physical volumes
- Logical Volumes (LV) which can be thought as a partition 

Note: You can create multiple volume groups, logical volumes, resize them in a matter that is more manageable than traditional partitions.

## Creating Physical Volumes, Volume Groups, and Logical Volumes
For this purpose I have a Virtual Machine setup with 3 disks of size 50Gb's <br>

To create physical volumes we use ``pvcreate`` <br>
```
[root@server ~]# pvcreate /dev/sdb /dev/sdc /dev/sdd
  Physical volume "/dev/sdb" successfully created.
  Physical volume "/dev/sdc" successfully created.
  Physical volume "/dev/sdd" successfully created.
[root@server ~]# 
```
We can list created physical volumes with ``pvs`` <br>
```
[root@server ~]# pvs
  PV         VG Fmt  Attr PSize  PFree 
  /dev/sdb      lvm2 ---  50.00g 50.00g
  /dev/sdc      lvm2 ---  50.00g 50.00g
  /dev/sdd      lvm2 ---  50.00g 50.00g
```
And get detailed information with ``pvdisplay`` <br>
```
[root@server ~]# pvdisplay /dev/sdb
  "/dev/sdb" is a new physical volume of "50.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb
  VG Name               
  PV Size               50.00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               IluKvB-JvI1-AEPk-cM9z-ZYo8-cwbH-8lGLb3
```
We can create volume group using ``vgcreate`` <br>
```
[root@server ~]# vgcreate testgroup /dev/sdb /dev/sdc /dev/sdd
  Volume group "testgroup" successfully created
```
Note: We cannot use one physical volume with multiple volume groups

Similarly, as with physical volumes we can use ``vgs`` and ``vgdisplay`` for information <br>
```
[root@server ~]# vgs 
  VG        #PV #LV #SN Attr   VSize    VFree   
  testgroup   3   0   0 wz--n- <149.99g <149.99g
[root@server ~]# vgdisplay testgroup
  --- Volume group ---
  VG Name               testgroup
  System ID             
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               <149.99 GiB
  PE Size               4.00 MiB
  Total PE              38397
  Alloc PE / Size       0 / 0   
  Free  PE / Size       38397 / <149.99 GiB
  VG UUID               78zgvU-fZcL-2FDI-Kg28-HfUW-xBdI-lr7OSG
```

To create logical volumes we use ``lvcreate`` <br>
```
[root@server ~]# lvcreate -n vol_backups -L 30G testgroup  
  Logical volume "vol_backups" created.
[root@server ~]# lvcreate -n vol_something -L 30G testgroup  
  Logical volume "vol_something" created.
```

We can think of them as virtual partitions, we can put filesystem on them and use them. With the benefit of having LVM on top of it for resizing and management <br>

We again have similar commands that provide us information ``lvs`` and ``lvdisplay`` <br>
```
[root@server ~]# lvs
  LV            VG        Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  vol_backups   testgroup -wi-a----- 30.00g                                                    
  vol_something testgroup -wi-a----- 30.00g                                                    
[root@server ~]# lvdisplay testgroup/vol_backups
  --- Logical volume ---
  LV Path                /dev/testgroup/vol_backups
  LV Name                vol_backups
  VG Name                testgroup
  LV UUID                Cq5Yqe-UA62-1Ehj-MksD-ML0e-DMLC-r7r6Lt
  LV Write Access        read/write
  LV Creation host, time server, 2025-07-16 14:07:01 +0000
  LV Status              available
  # open                 0
  LV Size                30.00 GiB
  Current LE             7680
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
   
[root@server ~]# 
```

We can now add ext4 or other filesystem to those LV's <br>
```
root@server ~]# mkfs.ext4 /dev/mapper/testgroup-vol_backups 
mke2fs 1.47.2 (1-Jan-2025)
Discarding device blocks: done                            
Creating filesystem with 7864320 4k blocks and 1966080 inodes
Filesystem UUID: 86bd631e-5d0c-4f1e-a9f9-848e3d8fa8ab
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
        4096000

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done   

[root@server ~]# mkfs.ext4 /dev/mapper/testgroup-vol_something 
mke2fs 1.47.2 (1-Jan-2025)
Discarding device blocks: done                            
Creating filesystem with 7864320 4k blocks and 1966080 inodes
Filesystem UUID: ba666542-c694-45e4-b6aa-2e0a42add967
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
        4096000

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done   

[root@server ~]# 
```

Which you can use as regular disks <br>
```
[root@server ~]# mount /dev/mapper/testgroup-vol_something /mnt
[root@server ~]# touch /mnt/testfile
```

## Resizing Logical Volumes and Extending Volume Groups
Before we can reduce the size of a LV we need to make the filesystem smaller <br>
Think about this way if filesystem is of size 100Gb's, and you will make the LV of size 50Gb <br>
You are effectively cutting 50Gb's and whatever data was on it, you don't want to perform a data loss <br>
In fact lvm will not allow you to do it unless you force it to <br>
```
[root@server ~]# lvreduce -L -2.5G testgroup/vol_something
  File system ext4 found on testgroup/vol_something.
  File system size (30.00 GiB) is larger than the requested size (27.50 GiB).
  File system reduce is required (see resize2fs or --resizefs.)
```

You can do it with ``resize2fs`` <br>
```
[root@server ~]# resize2fs /dev/mapper/testgroup-vol_something 27G
resize2fs 1.47.2 (1-Jan-2025)
Resizing the filesystem on /dev/mapper/testgroup-vol_something to 7077888 (4k) blocks.
The filesystem on /dev/mapper/testgroup-vol_something is now 7077888 (4k) blocks long.
```
Now we can resize the LV <br>
```
[root@server ~]# lvreduce -L -2.5G testgroup/vol_something
  File system ext4 found on testgroup/vol_something.
  File system size (27.00 GiB) is smaller than the requested size (27.50 GiB).
  File system reduce is not needed, skipping.
  Size of logical volume testgroup/vol_something changed from 30.00 GiB (7680 extents) to 27.50 GiB (7040 extents).
  Logical volume testgroup/vol_something successfully resized.
```
Note: Some filesystems do not allow resizing, allow only to extend it's size 

We can also increase the size of LV with <br>
```
[root@server ~]# lvextend -l +30%FREE testgroup/vol_something
  Size of logical volume testgroup/vol_something changed from 27.50 GiB (7040 extents) to 55.25 GiB (14144 extents).
  Logical volume testgroup/vol_something successfully resized.
```

In this case I increased the size by +30% available space in volume group <br>
After this operation make sure to increase your filesystem size (as otherwise the size will be the same) <br>
```
[root@server ~]# resize2fs /dev/mapper/testgroup-vol_something 
resize2fs 1.47.2 (1-Jan-2025)
Resizing the filesystem on /dev/mapper/testgroup-vol_something to 14483456 (4k) blocks.
The filesystem on /dev/mapper/testgroup-vol_something is now 14483456 (4k) blocks long.
```

After this operation, we can mount filesystem and we can see size was increased <br>

```
[root@server ~]# mount /dev/mapper/testgroup-vol_something /mnt
[root@server ~]# df -h
Filesystem                           Size  Used Avail Use% Mounted on
...
/dev/mapper/testgroup-vol_something   55G  2.1M   52G   1% /mnt
```

If we are running out of space in our volume group we can add new disk with ``vgextend testgroup /dev/(disk)`` <br>
And of course we can remove a disk from volume group ``vgreduce testgroup /dev/(disk)`` <br>
Note: If you want to remove disk where LV is already allocated you will have to use ``pvmove`` and then you can safely remove it from volume group 