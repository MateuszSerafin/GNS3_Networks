# Chapter 15: Kernel Runtime Parameters
In chapter 13 It was mentioned where to configure kernel command line parameters <br>
Note: Some parameters need to be specified at boot time such as when trying to do gpu passthrough <br>
Additionally the changing kernel parameters is really similar to different bootloaders <br>

## Introducing the /proc Filesystem
The ``/proc`` directory represents default way of handling process and system information as well as other kernel and memory information <br>
The ``/proc/sys`` directory is where you can find all the information about devices and drivers and some kernel features <br>

Inside of ``/proc/sys`` you can find <br>

| Folder | Description                                                     |
|--------|-----------------------------------------------------------------|
| dev    | Parameters for devices connected to the machine                 |
| fs     | filesystem configuration (for example quotas and inodes + more) |
| kernel | Kernel options                                                  |
| net    | Networking related                                              |
| vm     | Use of the kernel's virtual memory                              |

To set and view kernel runtime parameters we can use sysctl command <br>
```
[root@server ~]# sysctl -a
abi.vsyscall32 = 1
debug.exception-trace = 1
debug.kprobes-optimization = 1
dev.hpet.max-user-freq = 64
dev.mac_hid.mouse_button2_keycode = 97
dev.mac_hid.mouse_button3_keycode = 100
dev.mac_hid.mouse_button_emulation = 0
dev.parport.default.spintime = 500
dev.parport.default.timeslice = 200
dev.parport.parport0.base-addr = 888    0
dev.parport.parport0.devices.active = none
dev.parport.parport0.dma = -1
dev.parport.parport0.irq = 7
dev.parport.parport0.modes = PCSPP,TRISTATE
dev.parport.parport0.spintime = 500
dev.scsi.logging_level = 0
dev.tty.ldisc_autoload = 1
dev.tty.legacy_tiocsti = 0
fs.aio-max-nr = 65536
fs.aio-nr = 0
```
Note: sysctl is basically interacting with the special files located in ``/proc/sys`` directory <br>
We can take as an example ``fs.aio-max-nr = 65536`` which has path of ``/proc/sys/fs/aio-max-nr`` <br>
```
[root@server ~]# cat /proc/sys/fs/aio-max-nr 
65536
[root@server ~]# 
```
This means that using sysctl is exactly the same as interacting with those special files directly <br>

We can use ``sysctl -w (parameter)=(value)`` to change the values <br>
Alternatively we can use ``echo (value) > (kernel parameter file)`` which will do exactly the same thing <br>
```
[root@server ~]# cat /proc/sys/fs/aio-max-nr 
65536
[root@server ~]# sysctl -w fs.aio-max-nr=65535
fs.aio-max-nr = 65535
[root@server ~]# sysctl fs.aio-max-nr
fs.aio-max-nr = 65535
[root@server ~]# cat /proc/sys/fs/aio-max-nr 
65535
[root@server ~]# echo 65534 > /proc/sys/fs/aio-max-nr 
[root@server ~]# cat /proc/sys/fs/aio-max-nr 
65534
[root@server ~]# 
```

Although this configuration is not persistent meaning after reboot you will lose the configuration <br>
To make it persistent you can use ``sysctl -p`` but on arch linux wiki says to create a file under ``/etc/sysctl.d/(priority)-file.conf``
With for example <br>
```
[root@server ~]# cat /etc/sysctl.d/10-setaiomax.conf 
fs.aio-max-nr = 65535
```
So I guess it's a distro thing on how you need to do it <br>

As a note my system have 1012 kernel runtime parameters <br>
```
[root@server ~]# sysctl -a | wc -l
1012
```
Going through every single one is infeasible here are some examples what you can do with them <br>

| Parameter                     | Description                                                              |
|-------------------------------|--------------------------------------------------------------------------|
| net.ipv4.ip_forward           | Enables packet forwarding, or disables it (you can do the same for ipv4) |
| fs.file-max                   | Maximum number of file handles that kernel can allocate for the system   |
| kernel.sysrq                  | Determines if SysRq on your keyboard invokes emergency actions           |
| net.ipv4.icmp_echo_ignore_all | Determines if your system ignores icmp requests                          |

And much, much more ... 