# Chapter 19: The File Transfer Protocol (FTP)
Note: FTP doesn't support encryption, it still has its applications (As an example network switches and routers tend to use ftp) <br>
FTP has two modes <br>
- Active (Client initiates connection on port 21 and data is transferred over port 20)
- Passive (Client initiates connection on port 21, but also it handshakes with server to decide data port)

From what I read Passive FTP has fewer chances of being blocked by firewall on a network <br>

## Setting up an anonymous share
Let's start by installing ``vsftpd`` package <br>
```
[root@server ~]# pacman -S vsftpd
resolving dependencies...
looking for conflicting packages...

Packages (1) vsftpd-3.0.5-1

Total Download Size:   0.11 MiB
Total Installed Size:  0.27 MiB

:: Proceed with installation? [Y/n] y
:: Retrieving packages...
 vsftpd-3.0.5-1-x...   116.8 KiB  2.04 MiB/s 00:00 [#######################] 100%
(1/1) checking keys in keyring                     [#######################] 100%
(1/1) checking package integrity                   [#######################] 100%
(1/1) loading package files                        [#######################] 100%
(1/1) checking for file conflicts                  [#######################] 100%
(1/1) checking available disk space                [#######################] 100%
:: Processing package changes...
(1/1) installing vsftpd                            [#######################] 100%
Optional dependencies for vsftpd
    logrotate
:: Running post-transaction hooks...
(1/2) Reloading system manager configuration...
(2/2) Arming ConditionNeedsUpdate...
[root@server ~]# 
```

To configure we open up  ``/etc/vsftpd.conf`` file <br>
We will configure Anonymous share that points to ``/storage/anon`` <br>
To do this we just configure those lines <br>
```
anonymous_enable=YES
no_anon_password=YES
anon_root=/storage/anon/
write_enable=NO
```
Note: Allowing writes to anonymous share is unsafe additionally make sure that ``/storage/anon/`` is readable by user that runs ftp server <br>

Now when we start service we have a read only anonymous share <br>
```
[root@server ~]# systemctl start vsftpd
[root@server ~]# ftp 10.21.37.15
Connected to 10.21.37.15.
220 (vsFTPd 3.0.5)
Name (10.21.37.15:root): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0               7 Jul 19 23:27 asdasdas.txt
226 Directory send OK.
ftp> get asdasdas.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for asdasdas.txt (7 bytes).
226 Transfer complete.
7 bytes received in 0.0003 seconds (19.7184 kbytes/s)
ftp> put asdasdas.txt 
200 PORT command successful. Consider using PASV.
550 Permission denied.
ftp> rm asdasdas.txt 
550 Permission denied.
ftp> 
```

## Authenticating with local users
Alternatively we can authenticate with local users which will result in the same access to filesystem as local user <br>
```
local_enable=YES
write_enable=YES
```
```
[root@server storage]# ftp 10.21.37.15
Connected to 10.21.37.15.
220 (vsFTPd 3.0.5)
Name (10.21.37.15:root): testuser
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> pwd
257 "/home/testuser" is the current directory
ftp> mkdir hello
257 "/home/testuser/hello" created
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwx------    2 1001     1001         4096 Jul 20 21:55 hello
226 Directory send OK.
ftp> cd /root
550 Failed to change directory.
ftp> 
```

We can change default directory with <br>
```
user_sub_token=$USER
local_root=/storage/$USER/
```
Note: Make sure that ``/storage/(username)/`` exists and is readable/writable by ftp user <br>
Also there is nothing preventing you from just pointing to ``/storage/`` and it would make it a global directory for all ftp users <br>

```
[root@server ~]# ftp 10.21.37.15
Connected to 10.21.37.15.
220 (vsFTPd 3.0.5)
Name (10.21.37.15:root): testuser
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> pwd
257 "/storage/testuser" is the current directory
ftp> 
```

Or put chroot jail on it with <br>
```
chroot_local_user=YES
allow_writeable_chroot=YES
```

Basically it makes ftp user to see ``/storage/$USER`` directory as the only directory. It cannot traverse the system up from this directory <br>

```
[root@server ~]# ftp 10.21.37.15
Connected to 10.21.37.15.
220 (vsFTPd 3.0.5)
Name (10.21.37.15:root): testuser
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> pwd
257 "/" is the current directory
ftp> 
```

## Additional notes
We can configure additional things such as <br>
- Configure max upload/download rates for anonymous and regular users 
- Limit connections per ip
- Add a banner (message whenever user logs in)

Covering everything would be infeasible, what you should do is investigate if for your purpose ftp is good and configure it<br>