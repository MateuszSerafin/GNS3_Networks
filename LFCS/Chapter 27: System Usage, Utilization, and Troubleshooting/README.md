# Chapter 27: System Usage, Utilization, and Troubleshooting

## Storage Space Utilization
There are 2 well known command that can be used to inspect storage space usage (``df`` and ``du``) <br>
We can use ``df -h``, the ``-h`` option means to use human-readable units. <br> 
```
[root@server ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
dev             1.9G     0  1.9G   0% /dev
run             2.0G  660K  2.0G   1% /run
efivarfs        256K   43K  209K  17% /sys/firmware/efi/efivars
/dev/sda2       195G  4.3G  181G   3% /
tmpfs           2.0G     0  2.0G   0% /dev/shm
tmpfs           2.0G     0  2.0G   0% /tmp
tmpfs           1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
tmpfs           1.0M     0  1.0M   0% /run/credentials/systemd-resolved.service
tmpfs           1.0M     0  1.0M   0% /run/credentials/systemd-networkd.service
/dev/sda1      1022M  155M  868M  16% /boot
tmpfs           1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
tmpfs           391M  4.0K  391M   1% /run/user/0
```
As an alternative we can use ``du -h`` <br>
```
[root@server ~]# ls
asdasdas.txt  backup.img  data
[root@server ~]# du -h
8.0K    ./.gnupg/crls.d
12K     ./.gnupg
8.0K    ./.config/htop
12K     ./.config
4.0K    ./data
4.0K    ./.ssh
17M     .
[root@server ~]# 
```
Which will print the actual size of files in a current directory <br>
You can specify what directory should ``du`` check <br>
```
[root@server ~]# du -h /mnt
4.0K    /mnt
```
Alternatively we can use ``du`` to print information slightly differently <br>
```
[root@server ~]# du -sch /var/*
1.4G    /var/cache
20K     /var/db
4.0K    /var/empty
4.0K    /var/games
223M    /var/lib
4.0K    /var/local
0       /var/lock
33M     /var/log
0       /var/mail
16K     /var/named
4.0K    /var/opt
0       /var/run
20K     /var/spool
44K     /var/tmp
1.7G    total
```
Note: I am big fan of ``ncdu`` tool which is basically ``du`` but you can dynamically traverse directories and see what takes up space <br>

Aside of size with ``df`` we can check inode usage <br>
```
[root@server ~]# df -hTi
Filesystem     Type     Inodes IUsed IFree IUse% Mounted on
dev            devtmpfs   486K   483  486K    1% /dev
run            tmpfs      488K   703  488K    1% /run
efivarfs       efivarfs      0     0     0     - /sys/firmware/efi/efivars
/dev/sda2      ext4        13M   85K   13M    1% /
tmpfs          tmpfs      488K     1  488K    1% /dev/shm
tmpfs          tmpfs      1.0M    19  1.0M    1% /tmp
tmpfs          tmpfs      1.0K     1  1023    1% /run/credentials/systemd-journald.service
tmpfs          tmpfs      1.0K     1  1023    1% /run/credentials/systemd-resolved.service
tmpfs          tmpfs      1.0K     1  1023    1% /run/credentials/systemd-networkd.service
/dev/sda1      vfat          0     0     0     - /boot
tmpfs          tmpfs      1.0K     1  1023    1% /run/credentials/getty@tty1.service
tmpfs          tmpfs       98K    27   98K    1% /run/user/0
```

## Memory and CPU Utilization
We can use tools such as ``top`` and ``htop`` which are basically a task manager on a linux. (Let's not cross a line of explaining a task manager) <br>
Note: It's worth mentioning that we can use ps command to snapshot current processes and for example sort them by memory usage <br>
```
[root@server ~]# ps -eo user,comm,pid,ppid,%mem --sort -%mem
USER     COMMAND             PID    PPID %MEM
mysql    mariadbd            548       1  5.0
install+ dirmngr           16353       1  0.6
root     systemd-journal     257       1  0.4
root     systemd               1       0  0.3
systemd+ systemd-resolve     305       1  0.3
root     httpd             34464       1  0.2
root     systemd            3821       1  0.2
systemd+ systemd-network     318       1  0.2
http     httpd             34466   34464  0.2
root     sshd-session      44322     400  0.2
http     httpd             34467   34464  0.2
```
Or CPU usage <br>
```
[root@server ~]# ps -eo user,comm,pid,ppid,%cpu --sort -%cpu
USER     COMMAND             PID    PPID %CPU
root     kworker/u16:3-e   49024       2  0.1
root     kworker/2:0-mm_   44348       2  0.0
root     kworker/3:1-eve    3935       2  0.0
root     kworker/1:1-mm_    3895       2  0.0
root     kworker/0:2-mm_    3823       2  0.0
root     sshd-session      44328   44322  0.0
root     kworker/2:1-cgr      66       2  0.0
root     kworker/u16:2-e   41653       2  0.0
mysql    mariadbd            548       1  0.0
ntp      ntpd                410       1  0.0
root     httpd             34464       1  0.0
install+ keyboxd           16348       1  0.0
install+ gpg-agent         16358       1  0.0
```

Aside of that we can send signals to processes which was covered somewhere, we can easily do it with htop <br>
But What I want is a table <br>

| Signal  | Signal Number | Description                                                                                                       |
|---------|---------------|-------------------------------------------------------------------------------------------------------------------|
| SIGTERM | 15            | Kill the process gracefully                                                                                       |
| SIGINT  | 2             | This signal is equivalent of sending a ``CTRL+C`` on a keyboard, the aim is to interrupt the process              |
| SIGKILL | 9             | This signal also interrupts the process but the process cannot ignore it (effectively Linux shotguns the process) |
| SIGHUP  | 1             | Short for ``Hang UP`` instructs daemons to reread its configuration file without actually stopping the process    |
| SIGTSTP | 20            | Pause execution and wait ready to continue equivalent of ``Ctrl+Z`` on keyboard                                   |
| SIGSTOP | 19            | The process is paused and doesn't get more CPU cycles until it is restarted                                       |
| SIGCONT | 18            | Resumes execution (After SIGTSTP or SIGSTOP) equivalent of ``fg``,``bg`` command                                  |

## So… what Happened /is Happening?
After there was any kind of outage. ``/var/log`` is your friend. With it, you should be able to deduct what went wrong <br>
Additionally I highly suggest to use ``journalctl`` command which allows to look at systemd services history (in addition to ``/var/log``)

