# Chapter 7: Managing the Startup Process and Related Services

## Boot Process
1. POST (Power on Self Test), components are initializing and performing checks to determine if they are working correctly. 
2. In both BIOS and UEFI boot processes, the firmware performs additional checks before transferring control to the bootloader, which loads the Linux kernel into memory.
3. Bootloader loads the kernel and the initramfs
4. Kernel performs initializations
   - Initializes the file system
   - Sets up the networking stack
   - Mounts the root filesystem
   - Performs low-level initializations
5. Service manager initializes, starting various system services, including:
   - System logging
   - Device management
   - Network configuration
   - Docker
   - libvirt
   - SSH
   - Other services

Bullet point 3. reefers to grub but there are more Bootloaders https://wiki.archlinux.org/title/Arch_boot_process#Boot_loader <br>
Kernel is always initialized with PID 1. <br>
Kernel Parameters are passed with Bootloader. So if you want to do a gpu passthrough to VM you need to edit grub as an example <br>
Kernal has its own temporary filesystem with necessary tools after it's done, it mounts your disk. <br>
Some software operates at kernel level such as lvm, hence if you install it kernel will require rebuild of initramfs (to add new tools) and reboot is required. <br>

## Service Management
### SysV
For the purpose of this part I installed slackware. It's because it uses SysV <br>
From what I am reading its old not necessarily outdated but systemd is the successor <br>

From my understanding ``/etc/inittab`` controls to what "mode" do we boot <br>
Table below is from my study guide <br>

| Mode | Description                                                                                                                                                                                                                                                    |
|------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 0    | Halt the system. Runlevel 0 is a special transitional state used to shutdown the system quickly.                                                                                                                                                               |
| 1    | Also aliased to s, or S, this runlevel is sometimes called maintenance mode. What services, if any, are started at this runlevel varies by distribution. It’s typically used for low-level system maintenance that may be impaired by normal system operation. |
| 2    | Multiuser. On Debian systems and derivatives, this is the default runlevel, and includes -if available- a graphical login. On Red-Hat based systems, this is multiuser mode without networking.                                                                |
| 3    | On Red-Hat based systems, this is the default multiuser mode, which runs everything except the graphical environment. This runlevel and levels 4 and 5 usually are not used on Debian-based systems.                                                           |
| 4    | Typically, unused by default and therefore available for customization.                                                                                                                                                                                        |
| 5    | On Red-Hat based systems, full multiuser mode with GUI login. This runlevel is like level 3, but with a GUI login available.                                                                                                                                   |
| 6    | Reboot the system.                                                                                                                                                                                                                                             |

Again from my understanding <br>
For starting the service we would have something like <br>
```
/usr/sbin/sshd $SSHD_OPTS
```
This would be under ``rc.(your mode)/sshd``. When it would execute sshd would start in background <br>
To stop service we would have <br>
```
killall --ns $$ sshd-session 2> /dev/null
killall --ns $$ sshd
```
And it would be under ``rc.(0 and 6)/sshd`` <br>

We can use command ``chkconfig -list [service name]`` to check at what levels the service is enabled <br>
Or change them ``chkconfig -level [level(s)] service off`` <br>

For debian it would be "Sysv-rc-conf" with different syntax similar principle. <br>

Note: My study guide shown me two different distros, I installed slackware, and it's different from two mentioned <br>
I understand the basic concept behind it, I think its good enough. <br>

Note2: SysV is simple and starts services sequentially. 
### Systemd
#### Listing services
We can just type ``systemctl`` which will list all services and what is happening to them <br>
```      
...
  dbus-broker.service                                                                      loaded active running   D-Bus System Message Bus
  dirmngr@etc-pacman.d-gnupg.service                                                       loaded active running   GnuPG network certificate management daemon for /etc/pacman.d/gnup
g
  getty@tty1.service                                                                       loaded active running   Getty on tty1
  gpg-agent@etc-pacman.d-gnupg.service                                                     loaded active running   GnuPG cryptographic agent and passphrase cache for /etc/pacman.d/g
nupg
  kmod-static-nodes.service                                                                loaded active exited    Create List of Static Device Nodes
  ldconfig.service                                                                         loaded active exited    Rebuild Dynamic Linker Cache
...
```
We can also run ``systemctl status`` which will show tree of processes and some additional information <br>
```
● server
    State: running
    Units: 271 loaded (incl. loaded aliases)
     Jobs: 0 queued
   Failed: 0 units
    Since: Mon 2025-07-14 12:31:51 UTC; 18s ago
  systemd: 257.7-1-arch
   CGroup: /
           ├─init.scope
           │ └─1 /sbin/init
           ├─system.slice
           │ ├─dbus-broker.service
           │ │ ├─389 /usr/bin/dbus-broker-launch --scope system --audit
           │ │ └─390 dbus-broker --log 10 --controller 9 --machine-id 3ae97a2345bc48bdb9863c52a5082ed8 --max-bytes 536870912 --max-fds 4096 --max-matches 16384 --audit

           │ ├─sshd.service
           │ │ └─391 "sshd: /usr/bin/sshd -D [listener] 0 of 10-100 startups"
           │ ├─system-getty.slice
           │ │ └─getty@tty1.service
           │ │   └─401 /sbin/agetty -o "-- \\u" --noreset --noclear - linux
           │ ├─systemd-journald.service
           │ │ └─256 /usr/lib/systemd/systemd-journald
           │ ├─systemd-logind.service
           │ │ └─392 /usr/lib/systemd/systemd-logind
           │ ├─systemd-networkd.service
           │ │ └─321 /usr/lib/systemd/systemd-networkd
```
We can also do ``systemctl status (service)`` to check its current status in detail <br>
```
[root@server ~]# systemctl status sshd
● sshd.service - OpenSSH Daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; preset: disabled)
     Active: active (running) since Mon 2025-07-14 12:31:52 UTC; 2min 46s ago
 Invocation: be9a24459b1b4c3d99a82a8f40ae27e6
   Main PID: 391 (sshd)
      Tasks: 1 (limit: 4613)
     Memory: 8M (peak: 25.6M)
        CPU: 108ms
     CGroup: /system.slice/sshd.service
             └─391 "sshd: /usr/bin/sshd -D [listener] 0 of 10-100 startups"

Jul 14 12:31:52 server systemd[1]: Starting OpenSSH Daemon...
Jul 14 12:31:52 server sshd[391]: Server listening on 0.0.0.0 port 22.
Jul 14 12:31:52 server sshd[391]: Server listening on :: port 22.
Jul 14 12:31:52 server systemd[1]: Started OpenSSH Daemon.
Jul 14 12:32:03 server sshd-session[404]: Accepted password for root from 10.21.37.1 port 33400 ssh2
Jul 14 12:32:03 server sshd-session[404]: pam_unix(sshd:session): session opened for user root(uid=0) by root(uid=0)
```

#### Starting and Stopping services
```
[root@server ~]# systemctl stop sshd
[root@server ~]# systemctl start sshd
[root@server ~]# systemctl restart sshd
```
Note: Starting service this way is not persistent after reboot the service will still be down. 

#### Automatic starts
```
[root@server ~]# systemctl enable sshd
Created symlink '/etc/systemd/system/multi-user.target.wants/sshd.service' → '/usr/lib/systemd/system/sshd.service'.
[root@server ~]# systemctl disable sshd
Removed '/etc/systemd/system/multi-user.target.wants/sshd.service'.
[root@server ~]# 
```
We can also check if service is already marked for autostart <br>
```
[root@server ~]# systemctl is-enabled sshd
enabled
```

Note: If you enable service it will start after reboot. You still have to start in current session. This is also true for disabling <br>
Also not sure what else to include there it's really easy. 

#### Creating your own service
Systemd looks for services in those folders <br>

| Folder                   | Purpose                                     |
|--------------------------|---------------------------------------------|
| /etc/systemd/system/     | Your custom services                        |
| /usr/lib/systemd/system/ | Services from installed packages (like ssh) |
| ~/.config/systemd/user/  | User session services                       |

As an example of service I will use that <br>
```
/etc/systemd/system/gns3Container.service
[Unit]
[Description=gns3Container

[Service]
ExecStart=/usr/bin/systemd-nspawn -D /opt/gns3Container/ --boot --network-bridge=dev0 --bind=/dev/kvm
KillMode=mixed
```
Which just starts a container that runs GNS3 <br>
I can do ``systemctl start gns3Container`` or ``systemctl stop gns3Container`` which is quite handy <br>

Note: This is really simple example, with services you can do a lot.
- Limit CPU usage
- Limit Memory Usage
- Make service have a dependencies (such as mariadb if your service requires a database)
- Different types of services (such as oneshot) which will execute a command and that is it
- Services with timer that execute every 5 minutes, every weekend etc 
- Create a template of service that takes arguments ``systemctl start something@argument``

Worth taking a look https://www.freedesktop.org/software/systemd/man/latest/systemd.service.html