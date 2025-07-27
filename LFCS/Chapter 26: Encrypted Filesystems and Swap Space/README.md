# Chapter 26: Encrypted Filesystems and Swap Space
Before we start, my study guide suggest to wipe disk with something like ``dd if=/dev/urandom of=/dev/sdb bs=4096`` <br>
But this is highly depended on your situation, for example if you have a new disk, this would be pointless also this will reduce life of a ssd <br>

Note: Full disk encryption means that all of your data is encrypted (except boot partition) we still have to boot it somehow right <br>
Depending on situation we might not want to use Full disk encryption and perhaps only encrypt particular directories, https://wiki.archlinux.org/title/Data-at-rest_encryption worth a read

## Setting up an Encrypted Partition
The default tool that is used to manage encrypted disks on linux is ``cryptsetup``. <br>
To create an encrypted partition we will use following command <br>
```
[root@server ~]# cryptsetup -y luksFormat /dev/sdd

WARNING!
========
This will overwrite data on /dev/sdd irrevocably.

Are you sure? (Type 'yes' in capital letters): YES
Enter passphrase for /dev/sdd: 
Verify passphrase: 
[root@server ~]# 
```
It will ask you for password that you will use later to unlock, Don't forget it <br>
Note: cryptsetup automatically adjusts key sizes and algorithm underneath, depending on your ram and cpu speed <br>
You might want research more information about it and perhaps provide your own options (defaults are secure) <br>

After you created the encrypted partition we have open it with cryptsetup <br>
```
[root@server ~]# cryptsetup luksOpen /dev/sdd test
Enter passphrase for /dev/sdd: 
[root@server ~]# 
```
You will be requested a password, after providing the password you can use ``lsblk`` and you will see that encrypted block device will appear <br>
```
[root@server ~]# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
...  
sdd                         8:48   0   50G  0 disk  
└─test                    253:2    0   50G  0 crypt 
```
You are operating on ``/dev/mapper/(device name you provided when executing luksOpen command)`` and not on ``/dev/sdd`` <br>
At this point this is normal block device you can format with ext4 or whatever <br>
```
[root@server ~]# mkfs.ext4 /dev/mapper/test
mke2fs 1.47.3 (8-Jul-2025)
Creating filesystem with 13103104 4k blocks and 3276800 inodes
Filesystem UUID: 6122cb75-dace-477c-836f-0337854e2fe5
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
        4096000, 7962624, 11239424

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (65536 blocks): done
Writing superblocks and filesystem accounting information: done   

[root@server ~]# 
[root@server ~]# mount /dev/mapper/test /mnt
[root@server ~]# touch /mnt/testfile.txt
[root@server ~]# 
```
To umount the encrypted partition correctly you have to first umount the filesystem and then close it with cryptsetup <br>
```
[root@server ~]# umount /mnt
[root@server ~]# cryptsetup close test
[root@server ~]# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
fd0                         2:0    1    4K  0 disk 
sda                         8:0    0  200G  0 disk 
├─sda1                      8:1    0    1G  0 part /boot
└─sda2                      8:2    0  199G  0 part /
sdb                         8:16   0   50G  0 disk 
├─testgroup-vol_backups   253:0    0   30G  0 lvm  
└─testgroup-vol_something 253:1    0 55.3G  0 lvm  
sdc                         8:32   0   50G  0 disk 
└─testgroup-vol_something 253:1    0 55.3G  0 lvm  
sdd                         8:48   0   50G  0 disk 
[root@server ~]# 
```

Note: You might want to back up LUKS header as if you overwrite it there is no chance of recovery of your data <br>
```
[root@server ~]# cryptsetup luksHeaderBackup /dev/sdd --header-backup-file backup.img
[root@server ~]# cryptsetup luksDump backup.img 
LUKS header information
Version:        2
Epoch:          3
Metadata area:  16384 [bytes]
Keyslots area:  16744448 [bytes]
UUID:           1205c8e9-37ac-4e5c-b7c7-8b614ea85b89
Label:          (no label)
Subsystem:      (no subsystem)
Flags:          (no flags)

Data segments:
  0: crypt
        offset: 16777216 [bytes]
        length: (whole device)
        cipher: aes-xts-plain64
        sector: 512 [bytes]

Keyslots:
  0: luks2
        Key:        512 bits
```

## Encrypting the Swap Space for Further Security
Swap is used when your system is lacking memory it starts to swap files onto disk. Sometimes it might use swap to store things that were not accessed in ram for longer periods of time <br>
Assuming you have a database or something important in memory, and it would be written to swap. Well swap is not encrypted and someone could access this data <br>
We can setup partition that is encrypted and mkswap it and use it as swap which would be more secure <br>
(Swap was covered in one of the chapters) 

