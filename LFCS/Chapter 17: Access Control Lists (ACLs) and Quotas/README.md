# Chapter 17: Access Control Lists (ACLs) and Quotas
Access Control Lists (also known as ACLs) allow to define more grained control over files and directories over regular rwx permissions <br>

## Checking File System Compatibility With ACLs
We can check if acl is currently enabled using ``tune2fs`` command <br>
```
[root@server ~]# tune2fs -l /dev/sda2 | grep "Default mount"
Default mount options:    user_xattr acl
[root@server ~]# 
```
As we can see ``acl`` option is there <br>
If you don't see this option make sure to exclude ``noacl`` from your mount options <br>
And keep in mind that not all filesystems support acl <br>

## Setting ACLs
We can use ``getfacl`` to get information about current file <br>
```
[root@server testdir]# getfacl /testdir/testfile1.txt 
getfacl: Removing leading '/' from absolute path names
# file: testdir/testfile1.txt
# owner: root
# group: root
user::rw-
group::r--
other::r--

[root@server testdir]# 
```

We can use command ``setfacl`` to set permissions <br>
```
[root@server testdir]# setfacl -m u:testuser:rw /testdir/testfile1.txt 
[root@server testdir]# setfacl -m u:testuser1:r /testdir/testfile1.txt 
[root@server testdir]# getfacl /testdir/testfile1.txt 
getfacl: Removing leading '/' from absolute path names
# file: testdir/testfile1.txt
# owner: root
# group: root
user::rw-
user:testuser:rw-
user:testuser2:r--
group::r--
mask::rw-
other::r--
```
As you can the difference between regular permissions, we can specify exact number of users we want and groups and what they can do <br>

Here are some examples <br>

| Command                    | Description                                                                                        |
|----------------------------|----------------------------------------------------------------------------------------------------|
| setfacl -m d:o:r /testdir/ | Sets a default permission for group "other" of read, the directories will inherit this permissions |
| setfacl -x d:o /testdir/   | Removes acl for "others" from ``/testdir/``                                                        |
| setfacl -b /testdir/       | Removes all acl's from ``/testdir/`` (regular linux permission are left over)                      |

Note: ACLs are basically Regular Linux permissions but per group and user (You can specify more than one user and group where normally you have just Owner, Group and Other)

## Disk Quotas
Perhaps we might want to limit space per group or per user as some people might get idea to fill it up to full because they can <br>

Before we begin we need to mount filesystem with option ``usrquota`` or ``grpquota`` <br>
```
[root@server testdir]# mount /dev/mapper/testgroup-vol_something /mnt -o usrquota
```
Additionally, we have to turn the quotas on <br>
```
[root@server testdir]# quotaon -vu /mnt
quotaon: Your kernel probably supports ext4 quota feature but you are using external quota files. Please switch your filesystem to use ext4 quota feature as external quota files on ext4 are deprecated. You can enable the feature by unmounting the file system and running 'tune2fs -O quota <device>'.
/dev/mapper/testgroup-vol_something [/mnt]: user quotas turned on
```
Note: The command for group rather than users would be ``quotaon -vg /home/projects`` <br>

At this point we are to set quotas <br>
```
#Let's give permission to user 
[root@server testdir]# setfacl -m u:testuser:rwx /mnt
[root@server testdir]# edquota -u testuser
```
Similarly, as with crontab when you execute ``edquota`` command you will be presented with your text editor where you set options for your selected user (or group if you specify ``-g`` option) <br>
```
Disk quotas for user testuser (uid 1001):
  Filesystem                   blocks       soft       hard     inodes     soft     hard
  /dev/mapper/testgroup-vol_something          0          0          0          0        0        0
```
We can specify following options <br>

| Option       | Description                                                                                      |
|--------------|--------------------------------------------------------------------------------------------------|
| Blocks       | Shows current number of blocks used by user or group on that filesystem                          |
| Soft         | Soft limit for disk message (inblocks, if user exceeds this limit a warning will appear)         |
| Hard         | The hard limit for disk usage (in blocks) Once this limit is reached user cannot write more data |
| inodes       | Shows current number of inodes used by a user or group                                           |
| soft (again) | Soft limit for inodes                                                                            |
| hard (again) | Hard limit for inodes                                                                            |

I will create a soft limit of blocks of 900 and hard limit of 1000 <br>
Which will be around 1MB which is good enough for testing <br>
```
Disk quotas for user testuser (uid 1001):
  Filesystem                   blocks       soft       hard     inodes     soft     hard
  /dev/mapper/testgroup-vol_something          0        900       1000          0        0        0
```
Now when I log in to user and try to write data <br>
```
[root@server testdir]# sudo -i -u testuser
[testuser@server ~]$ cd /mnt/
[testuser@server mnt]$ ls
aquota.user  lost+found  testfile
[testuser@server mnt]$ dd i^C
[testuser@server mnt]$ dd if=/dev/urandom of=aaa.test
dd: writing to 'aaa.test': Disk quota exceeded
2001+0 records in
2000+0 records out
1024000 bytes (1.0 MB, 1000 KiB) copied, 0.00637509 s, 161 MB/s
[testuser@server mnt]$ 
```

I got message that disk quota exceeded let's try increasing it by 0 <br>
```
[testuser@server mnt]$ dd if=/dev/urandom of=aaa.test
dd: writing to 'aaa.test': Disk quota exceeded
20001+0 records in
20000+0 records out
10240000 bytes (10 MB, 9.8 MiB) copied, 0.0797232 s, 128 MB/s
```
Cool exactly as expected <br>

Note: For groups behaviour would be the same 

Note2: We can use ``edquota -t`` to configure when soft limits should be enforced <br>
```
Grace period before enforcing soft limits for users:
Time units may be: days, hours, minutes, or seconds
  Filesystem             Block grace period     Inode grace period
  /dev/mapper/testgroup-vol_something                  7days                  7days
```

To view current quotas we can use command ``repquota`` or ``quota -u [user]`` or ``quota -g [group]`` <br>
```
[root@server testdir]# repquota -v /mnt
*** Report for user quotas on device /dev/mapper/testgroup-vol_something
Block grace time: 7days; Inode grace time: 7days
                        Block limits                File limits
User            used    soft    hard  grace    used  soft  hard  grace
----------------------------------------------------------------------
root      --    2068       0       0              4     0     0       
testuser  +-   10000     900   10000  6days       1     0     0       

Statistics:
Total blocks: 7
Data blocks: 1
Entries: 2
Used average: 2.000000

[root@server testdir]# quota -u testuser
Disk quotas for user testuser (uid 1001): 
     Filesystem  blocks   quota   limit   grace   files   quota   limit   grace
/dev/mapper/testgroup-vol_something
                  10000*    900   10000   6days       1       0       0        
[root@server testdir]# 
```

Note which i forgot about: Some file systems such as btrfs provide its own internal quoting system. 