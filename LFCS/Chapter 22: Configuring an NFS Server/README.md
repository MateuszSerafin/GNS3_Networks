# Chapter 22: Configuring an NFS Server
As part of Chapter 5 I set up an NFS server, but In this chapter I will go in depth <br>
Note: You can configure LDAP or NIS for authentication but LFCS does not require it 

## Exporting Network Shares
In ``/etc/exports`` you specify what directories should be shared to what clients <br>
The format of this configuration file looks as follows <br>
```
/filesystem/to/export client1([options]) clientN([options])
```
As a working example I will use <br>
```
/opt/nfs 10.21.37.0/24(rw)
```
Which shares ``/opt/nfs`` directory to clients on a network 10.21.37.0/24 with permissions of read and write <br>
We can specify multiple clients with different options <br>
```
/opt/nfs 10.21.37.5(rw) 10.21.37.7(ro)
```
Here is a table with available options <br>

| Option            | Description                                                                                                                   |
|-------------------|-------------------------------------------------------------------------------------------------------------------------------|
| ro                | Read only                                                                                                                     |
| rw                | Read-write                                                                                                                    |
| wdelay            | Delay between writing on a disk (No idea why you would need this option)                                                      |
| sync              | NFS Server makes sure that data is written to disk before handling more files                                                 |
| async             | Opposite of sync, NFS Server tries to manage files asynchronously use caching ( it may say it's saved but perhaps not yet)    |
| root_squash       | Prevents remote root users from having superuser privileges (assigns them with group nobody)                                  |
| all_squash        | Opposite of root_squash                                                                                                       |
| anonuid / anongid | Sets the UID and GID of the anonymous account (nobody)                                                                        |
| subree_check      | If only a subdirectory of a file system is exported, it verifies that requested file is located in that exported subdirectory |
| no_subtree_check  | If whole filesystem is exported we can use this option to speed up transfers                                                  |

After configuring ``/etc/exports`` and starting the service (``nfs-server`` in case of Arch Linux) we can check currently exported directories with <br>
```
[root@server ~]# showmount -e
Export list for server:
/opt/nfs 10.21.37.0/24
```

## Mounting Exported Network Shares Using Autofs
Mounting via fstab was covered in chapter 5 <br>

As an alternative to fstab, we can use Autofs, which mounts the network share only if it's needed. If the share becomes inactive for certain amount of time (configurable) it will be unmounted and our system will use less resources <br>

After installing autofs package <br>
We can configure it, It has a configuration file under ``/etc/auto.master`` (on Arch Linux ``/etc/autofs/auto.master``) <br>
In there we configure a line with directory where mounts will be located, config for the mount (which is a separate file that we have to create) and timeout option <br>
```
/mnt /etc/autofs/auto.file_server -timeout=60
```
Additionally, the ``/etc/autofs/auto.file_server`` file <br>
```
foo  -rw,soft,rsize=8192,wsize=8192 10.21.37.15:/opt/nfs
```
Note: the first field (``foo``) is the actual directory where this mount will be located in this case ``/mnt/foo/`` <br>

At this point we can start the service <br>
```
[root@client ~]# systemctl start autofs
[root@client ~]# 
```

Note: When you will ls directory you will see nothing <br>
```
[root@client ~]# ls /mnt
[root@client ~]# 
```
You will be also unable to do anything in this directory <br>
```
[root@client ~]# mkdir /mnt/asdasd
mkdir: cannot create directory ‘/mnt/asdasd’: Permission denied
```
It will only appear when you specifically try to access this share <br>
```
[root@client etc]# ls -al /mnt
total 4
drwxr-xr-x  2 root root    0 Jul 22 21:34 .
drwxr-xr-x 19 root root 4096 Jul 22 21:34 ..
[root@client etc]# ls -al /mnt/foo
total 8
drwxrwxrwx 3 root   root   4096 Jul 21 20:52 .
drwxr-xr-x 3 root   root      0 Jul 22 21:34 ..
drwxr-xr-x 2 nobody nobody 4096 Jul 21 20:52 tonapeno
[root@client etc]# ls -al /mnt
total 8
drwxr-xr-x  3 root root    0 Jul 22 21:34 .
drwxr-xr-x 19 root root 4096 Jul 22 21:34 ..
drwxrwxrwx  3 root root 4096 Jul 21 20:52 foo
[root@client etc]# 
```
Additionally, after timeout it will "hide" again (unmount) <br>

