# Chapter 16: SELinux and AppArmor
SELinux and AppArmor allow for additional security, they allow to restrict processes access to system objects, network controls and much more <br>

Note: SELinux and AppArmor are two different pieces of software which aim for similar goal <br>
Additionally MAC stands for Mandatory Access Control 

## SELinux
SELinux operates in two ways <br>
- Enforcing - SELinux denies access based on SELinux policy rules
- Permissive - SELinux does not deny access, but denials are logged for actions that would have been denied if running in enforcing mode 

We can check if selinux is enabled under ``/etc/selinux/config`` <br>
Which will have a line containing its current mode (or if it's disabled) <br>
```
SELINUX=enforcing
```

Alternatively we can use ``sestatus`` command <br>
```
root@localhost:/etc/ssh# sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Memory protection checking:     actual (secure)
Max kernel policy version:      33
```

My study guide shows me two examples where I might need to interact with selinux <br>
1. Changing the default port where a daemon listens on <br>
    As an example I will use sshd service, upon changing its port in ``/etc/ssh/sshd_config`` <br>
    And trying to restart the service it just doesn't work <br>
    ```
    root@localhost:/etc/ssh# systemctl restart sshd
    Job for sshd.service failed because the control process exited with error code.
    See "systemctl status sshd.service" and "journalctl -xeu sshd.service" for details.
    ```
    We can check ``/var/log/audit/audit.log`` which will confirm it was selinux that prevented this action <br>
    ```
    root@localhost:/etc/ssh# cat /var/log/audit/audit.log | grep tcp_socket
    type=AVC msg=audit(1752842580.654:437): avc:  denied  { name_bind } for  pid=14017 comm="sshd" src=9999 scontext=system_u:system_r:sshd_t:s0-s0:c0.c1023 tcontext=system_u:object_r:jboss_management_port_t:s0 tclass=tcp_socket permissive=0
    ```
    Using ``semanage`` command we can adjust on what port sshd will be <br>
    ```
    root@localhost:/etc/ssh# semanage port -m -t ssh_port_t -p tcp 9999
    root@localhost:/etc/ssh# systemctl restart sshd
    root@localhost:/etc/ssh#
    ```
    Service started without issues <br>
    
    Note: We can use ``semanage port -l`` to list what ports are assigned to services <br>
    ```
    root@localhost:/etc/ssh# semanage port -l
    SELinux Port Type              Proto    Port Number
    
    afs3_callback_port_t           tcp      7001
    afs3_callback_port_t           udp      7001
    afs_bos_port_t                 udp      7007
    afs_fs_port_t                  tcp      2040
    afs_fs_port_t                  udp      7000, 7005
    afs_ka_port_t                  udp      7004
    afs_pt_port_t                  tcp      7002
    afs_pt_port_t                  udp      7002
    afs_vl_port_t                  udp      7003
    agentx_port_t                  tcp      705
    agentx_port_t                  udp      705
    amanda_port_t                  tcp      10080-10083
    amanda_port_t                  udp      10080-10082
    amavisd_recv_port_t            tcp      10024
    amavisd_send_port_t            tcp      10025
    amqp_port_t                    tcp      15672, 5671-5672
    amqp_port_t                    udp      5671-5672
    ...
    ```
2. Webserver <br>
   By default SELinux expects that services like apache will read from directory ``/var/www/html`` <br>
   If you change the directory of the webserver to ``/websrv/`` or an attacker would want to snoop around for other files <br>
   SELinux will deny access as this folder is unexpected to be used <br>
   Note: I logged in into the user and user itself still has access to the file so this is relevant to the service itself only (perhaps you can configure it) <br>
   As you can see we get the expected message from SELinux <br> 
   ```
   root@localhost:/# cat /var/log/audit/audit.log | grep AVC | tail -1
   type=AVC msg=audit(1752844452.739:608): avc:  denied  { getattr } for  pid=14701 comm="httpd" path="/websrv/index.html" dev="dm-0" ino=69185539 scontext=system_u:system_r:httpd_t:s0 tcontext=unconfined_u:object_r:default_t:s0 tclass=file permissive=0
   ```
   We can use following command to add the directory <br>
   ```
   root@localhost:/# semanage fcontext -a -t httpd_sys_content_t "/websrv(/.*)?"
   ```
   Which will allow me to access the website normally <br>

## AppArmor
AppArmor has exact the same modes, but the naming is different <br>

## Note
This Chapter requires a revisit when I have more time
