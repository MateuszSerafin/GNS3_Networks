# Chapter 18: Setting up Network Services
Note: The more network services your machine is running the bigger it's attack surface. You might want to consider using virtual machines or docker (or some other isolation) <br>
Note2: My study guide in this chapter just installs NFS, Squid + SquidGuard + Apache without configuring anything <br>
This is especially weird where it looks like chapter 22,23 could cover it <br>

The only bullet point that mentions configuration of anything is NTP <br>
## Synchronizing time Using a Network Time Protocol (NTP) Server
1. Install ntp package <br>
   - Ubuntu - ``apt install ntpdate``
   - Centos - ``yum install ntpdate``
   - Arch Linux - ``pacman -S ntp`` 
2. Use ``ntpdate`` update time with ntp server <br>
   ```
   [root@server testdir]# ntpdate 1.ro.pool.ntp.org
   18 Jul 22:30:36 ntpdate[4969]: adjust time server 86.127.71.168 offset +0.000499 sec
   ```
3. Alternatively configure ``/etc/ntp.conf`` and enable ``ntpd`` service to keep your system time in sync <br>
   ```
    [root@server testdir]# cat /etc/ntp.conf 
    # Please consider joining the pool:
    #
    #     http://www.pool.ntp.org/join.html
    #
    # For additional information see:
    # - https://wiki.archlinux.org/index.php/Network_Time_Protocol_daemon
    # - http://support.ntp.org/bin/view/Support/GettingStarted
    # - the ntp.conf man page
    
    # Associate to Arch's NTP pool
    server 0.arch.pool.ntp.org
    server 1.arch.pool.ntp.org
    server 2.arch.pool.ntp.org
    server 3.arch.pool.ntp.org
    
    # By default, the server allows:
    # - all queries from the local host
    # - only time queries from remote hosts, protected by rate limiting and kod
    restrict default kod limited nomodify nopeer noquery notrap
    restrict 127.0.0.1
    restrict ::1
    
    # Location of drift file
    driftfile /var/lib/ntp/ntp.drift
    [root@server testdir]# systemctl --now enable ntpd 
    Created symlink '/etc/systemd/system/multi-user.target.wants/ntpd.service' → '/usr/lib/systemd/system/ntpd.service'.
    [root@server testdir]#
    ```
   

Because this chapter was really short, I will explain NTP more in depth as I feel like leaving that in that state is insufficient <br>
Let's start with why NTP is needed, pretty much all of modern encryption requires systems clock to be up to date <br>
This is because certificates rely on dates, some encryption relies on time as part of authenticity guarantor, (imagine if other computer gives you key that is valid to X time if your time is incorrect either you or other computer might say that this time is invalid and further communication just won't occur) <br>
Additionally the hardware clock that your computer has will go out of sync. It's pretty cheap comparatively to what big companies have, they will have a atomic clock worth millions of dollars that will keep time up to nanoseconds (where your computer might drift seconds daily) <br>
NTP is Divided into stratum levels <br>
Stratum 0 would be someone with really good atomic clock, like google <br>
Stratum 1 would be your ISP that would use Google's atomic clock to provide information about current time to it's users <br>
Stratum 2 could be your router, but that would be pretty rare <br>
The reason why it's divided is so you and everyone else would not effectively DDOS Stratum 0 servers additionally you do not require your computer clock to be perfectly in sync, however the time keeping is important for example for scientific computing <br>
Note: With each stratum level the accuracy of time decreases <br>
Additionally you can set up your own NTP server and keep your local network up to do date by your self, you could point it to google <br>
Effectively <br>
Stratum 0 would be google <br>
Stratum 1 would be your NTP server which is pretty cool <br>