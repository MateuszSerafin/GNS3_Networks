# Chapter 29: Network Performance, Security, and Troubleshooting

## What Services are Running and Why?
We do not want to run unnecessary services as it just increases the attack vectors on our machine <br>
For example If we are running FTP server, but it's not needed why leave it running and risk someone accessing our files <br>

Note: Management of services was covered under Chapter 7

## Investigating Socket Connections with ss
We can use command ``ss`` (Replacement of netstat) which will list current connections to our machine or what services are listening on what ports (there is more) <br>
```
root@localhost:~# ss
Netid           State            Recv-Q           Send-Q                                                    Local Address:Port                          Peer Address:Port            
u_str           ESTAB            0                0                                           /run/dbus/system_bus_socket 17648                                    * 16841           
u_str           ESTAB            0                0                                           /run/systemd/journal/stdout 9789                                     * 9788            
u_str           ESTAB            0                0                                                                     * 19297                                    * 19298           
u_dgr           ESTAB            0                0                                                                     * 19295                                    * 288             
u_str           ESTAB            0                0                                           /run/systemd/journal/stdout 7926                                     * 7005   
```
We can for example filter for tcp connections <br>
```
root@localhost:~# ss -t
State                  Recv-Q                  Send-Q                                     Local Address:Port                                     Peer Address:Port                   
ESTAB                  0                       52                                           10.21.37.11:ssh                                        10.21.37.1:51108   
```
Or filter for all programs that are listening on whatever port <br>
```
root@localhost:~# ss -l
Netid          State           Recv-Q          Send-Q                                                Local Address:Port                                   Peer Address:Port          
nl             UNCONN          0               0                                                              rtnl:NetworkManager/929                                 *              
nl             UNCONN          0               0                                                              rtnl:kernel                                             *              
nl             UNCONN          0               0                                                              rtnl:NetworkManager/929                                 *              
nl             UNCONN          4352            0                                                           tcpdiag:ss/25227                                           *              
nl             UNCONN          896             0                                                           tcpdiag:kernel                                             *              
nl             UNCONN          0               0                                                              xfrm:kernel                                             *              
nl             UNCONN          0               0                                                           selinux:kernel                                             *              
nl             UNCONN          0               0                                                             audit:dbus-broker/877                                    *              
nl             UNCONN          0               0                                                             audit:kernel                                             *              
nl             UNCONN          0               0                                                             audit:auditd/830                                         *              
nl             UNCONN          0               0                                                             audit:systemd/1                                          *              
nl             UNCONN          0               0                                                         fiblookup:kernel                                             *              
nl             UNCONN          0               0                                                         connector:kernel                                             *              
nl             UNCONN          0               0                                                               nft:systemd/1                                          *              
nl             UNCONN          0               0                                                               nft:firewalld/880                                      *              
nl             UNCONN          0               0                                                               nft:kernel                                             *   
```
I use it rarely, but it's useful to know <br>

## nmap
Nmap is used to scan ports, it will show you what services are running at what ports <br>
I will not go too much in depth <br>
But the tldr from security perspective it's good to scan your own machines to verify that you did not leave a service without correct firewall or just forgot to turn it off <br>

For example, using ``-F`` option will perform a scan over most common ports, In my case it's only ssh that is enabled and I am happy with the result <br>
```
root@localhost:~# nmap -F 10.21.37.1
Starting Nmap 7.92 ( https://nmap.org ) at 2025-07-25 16:29 BST
Nmap scan report for 10.21.37.1
Host is up (0.00034s latency).
Not shown: 99 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: 22:FA:47:63:D7:6C (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.16 seconds
```

## Reporting Usage and Performance on Your Network
You can use ``nload``, ``vnstat``, ``nmon`` <br>
Not sure what else to mention there we want to keep track of the amount of bandwidth we use weather it impacts users or not weather we have spikes in traffic and what is their reason <br>

Note: vnstat collects data hourly and plots it in a graph where nload and nmon is more of what currently is happening to our network interfaces and processes <br>

## Transferring Files Securely Over the Network
sftp or ncp. Both are provided by ssh server. Syntax is similar to ftp <br>
```
root@localhost:~# sftp root@10.21.37.15
The authenticity of host '10.21.37.15 (10.21.37.15)' can't be established.
ED25519 key fingerprint is SHA256:oOv16zlCTT8c9lrqYYyJkj1w4kiVBw52oH9otilMyqs.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.21.37.15' (ED25519) to the list of known hosts.
root@10.21.37.15's password: 
Connected to 10.21.37.15.
sftp> ls
\dir           asdasdas.txt   backup.img     data           
sftp> get backup.img
Fetching /root/backup.img to backup.img
backup.img                                                                                                                                         100%   16MB  59.8MB/s   00:00    
sftp> mkdir test
sftp> mv backup.img te
```
(Yea, moving files over network is not a breathtaking moment, although rsync is cool)

## Configuring SSH Servers and Clients
1. You want to use SSH keys
2. Restrict users that should be allowed to ssh 
3. On top of it additional security such as properly configured sudoers

