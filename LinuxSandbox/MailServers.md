# MailServers
I needed mail server for packetfence, then I realised I never did it. <br>
There is no particular scenario rather milestones. <br>
It's because I started configuring it and then after 2 days I kinda was not progressing anywhere. <br>

## Milestones
1. Configure basic mail no authentication no encryption it must just work ✅
2. Configure virtual users ✅
3. Configure sasl authentication ❌  
4. Reconfigure everything so it actually supports encryption and is safe to use ❌ 
5. Configure Exchange Server (So it works with ADDS) ❌ 
6. Configure a nice solution https://wiki.archlinux.org/title/Virtual_user_mail_system_with_Postfix,_Dovecot_and_Roundcube (Won't be done in foreseeable future) ❌ 

## Network Topology
![](MailServers/Topology.png) <br>
1. There is no reason why ThunderBirdClient is where it is (it just needs to access all the MailServers)
2. This network only provides routing, so I can test dns and mail servers correctly the network design behind it is not existent.                 

## DNS Configuration
1. First device added to this network was DNS. This is plain arch linux running "named" dns server.
2. For each domain used (CompanyA.A, CompanyB.B, CompanyC.C). I created a forward zone and reverse zone in /etc/named.conf <br>
    ```
    zone "CompanyA.A" {
        type master;
        file "CompanyA.A.zone";
    };
    
    zone "0.168.192.in-addr.arpa" IN {
        type master;
        file "192.168.0.zone";
    };
    ```
3. For each domain I then created the zone files. Forward zone required A, NS and MX record. (Each domain was the same config just different ip addresses) <br>
    ```
   $TTL 604800
    @       IN      SOA     ns.CompanyA.A. admin.CompanyA.A. (
                            2024052001     ; Serial
                            3600           ; Refresh
                            1800           ; Retry
                            604800         ; Expire
                            86400          ; Minimum TTL
    )
    
    @       IN      NS      ns.CompanyA.A.
    ns      IN      A       8.8.8.8
    @       IN      MX      10         mail.CompanyA.A.
    mail    IN      A       192.168.0.2
   ```
4. Additionally for reverse zone it looks like that. <br>
    ```
   $TTL 604800
    @   IN  SOA  ns.CompanyA.A. admin.CompanyA.A. (
                    2024052001 ; Serial
                    3600       ; Refresh
                    1800       ; Retry
                    604800     ; Expire
                    86400      ; Minimum TTL
    )
    
    @       IN  NS  ns.CompanyA.A.
    2     IN  PTR mail.CompanyA.A.
   ```
5. Sample Dig records for CompanyA.A are available in MailServers/DigSamples.txt (170 lines there would mess this document) <br>


Additionally, this could have been configured to CompanyA.com, CompanyB.com there is no reason why I did this way, but I just rolled with it <br>
Also while progressing and researching that somewhat recently it's required to have SPF and some more additional txt records in DNS. If you don't have that gmail could mark emails as spam. <br>
This is good to know however for the purpose of this lab this is N/A. <br>
https://gist.github.com/howyay/57982e6ba9eedd3a5662c518f1b985c7
https://docker-mailserver.github.io/docker-mailserver/latest/usage/

## Milestone 1
While setting this up I had no idea what to do. So I cheated and I used docker (I needed a known good configuration, so I can debug what I am doing). <br>
CompanyA.A and CompanyB.B are running https://docker-mailserver.github.io/docker-mailserver/latest/ <br>
This configuration is still invalid as I believe end-to-end encryption is not enabled and this is bare minimum configuration (which is insecure). <br>
In milestone 4 I will reconfigure it so it would be "production ready". However, to just test my configuration it's good enough.

1. I installed arch for CompanyA and CompanyB, configured addressing installed docker, not interesting stuff.
2. Copy compose.yaml from their github. I provide mine in MailServers/Milestone1. The only thing i changed is hostname to mail.CompanyA.A (and mail.CompanyB.B) for B <br>
3. Again copy mailserver.env from their github. I didn't do anything to it.
4. Run "docker compose up -d"
5. Run "docker exec -ti (container name) setup email add (user@domain) (password)"

At this point you can connect with thunderbird or other email client with the credentials you just put. <br>
Additionally sending emails between CompanyA.A and CompanyB.B works. <br>
![](MailServers/Milestone1/WorkingATOB.png) <br>

----
Yea at this point starts manual configuration of postfix and dovecot. Took me some time to understand how this works. <br>
It's again important to mention that this configuration is insecure and I will configure it in future to be secure.
1. Installed fresh arch linux configured it with hostname mail.CompanyC.C etc. installed postfix dovecot 
2. Configure postfix. <br>
    `/etc/postfix/main.cf`
    ```
    # This is where emails are saved, this will be located in user linux home directory. If it's file all emails are saved to one file
    # If it's directory everything is nicely split into directories. (Dovecot supports both formats, it's important to configure dovecot to read this folder file as otherwise you might be able to send email and receive but it won't be visible in your mailbox)
    home_mailbox = MailDir/
   
    #Those are variables
    myhostname = mail.companyc.c
    mydomain = companyc.c
    
    #Origin of email for example this server could receive from CompanyA.A and CompanyB.B however it will respond with $mydomain domain
    myorigin = $mydomain
    
    #This is actually what emails it will handle for example removing $mydomain (which evaluates to companyc.c) means it just wont handle @companyc.c domains.
    mydestination = $myhostname localhost.$mydomain localhost $mydomain
    
    #on what interfaces to listen
    inet_interfaces = all
    
    #This setting is big security risk it's essentially authentication. It allows hosts from certain networks to use our server
    #However by using 0.0.0.0/0 it means everyone can use this server to forward email which is not particularly great (For initial configuration and lack of my understanding it was good)
    #Later on this is disabled and SASL authentication handles whether host can send mail or not
    mynetworks = 0.0.0.0/0

    # This setting is responsible for forwarding mail. In this case it only forwards mails to my domain. Which is a secure option
    relay_domains = $mydomain
    # Specifies whether other host is responbile for forwarding our mail, with this setting our current server handles forwards
    relayhost =
    ```
3. For dovecot, I followed all steps from https://wiki.archlinux.org/title/Dovecot. From 2.2 to 2.5  <br>
    - Create TLS certificate (This means that accessing mailbox is encrypted, but it doesn't mean the sending or receiving is, as postfix needs to be configured for that, neither it's end-to-end encrypted) <br>
        ``cp /usr/share/doc/dovecot/dovecot-openssl.cnf /etc/ssl/dovecot-openssl.cnf`` <br>
        Edit ``/etc/ssl/dovecot-openssl.cnf`` <br>
        This actually generates the TLS certificate <br>
        ``./usr/lib/dovecot/mkcert.sh`` <br>
        ``cp /etc/ssl/certs/dovecot.pem /etc/ca-certificates/trust-source/anchors/dovecot.crt`` <br>
        Updates system certificates <br>
        ``trust extract-compat``
    - Copy default dovecot configurations <br>
        ``mkdir /etc/dovecot`` <br>
        ``cp /usr/share/doc/dovecot/example-config/dovecot.conf /etc/dovecot/dovecot.conf`` <br>
        ``cp -r /usr/share/doc/dovecot/example-config/conf.d /etc/dovecot``
    - Generate DH Key <br>
        ``openssl dhparam -out /etc/dovecot/dh.pem 4096`` <br>
        and configure <br>
        ``/etc/dovecot/conf.d/10-ssl.conf`` <br>
        with line <br>
        ``ssl_dh = </etc/dovecot/dh.pem``
4. Configure Mail Dir (For postfix default format is file, but I changed it to dir this needs to reflect dovecot) <br>
    Edit ``/etc/dovecot/conf.d/10-mail.conf`` <br>
    And place config <br>
    ``mail_location = maildir:~/Maildir``
5. Configure PAM authentication <br>
    Edit ``/etc/dovecot/conf.d/auth-system.conf.ext`` <br>
    And place/uncomment <br>
    ```
   passdb {
      driver = pam
      # [session=yes] [setcred=yes] [failure_show_msg=yes] [max_requests=<n>]
      # [cache_key=<key>] [<service name>]
      args = session=yes dovecot
    }
    ```
   
At this point when you create user <br>
``useradd -m amongas`` <br>
``passwd amongas`` <br>
You will be able to log in via thunderbird or other email client and send and receive emails. <br>
The problem is that there is no security however for milestone one it was good enough.
    
## Milestone 2
Assume you have lots of users. Having to do linux user for each one of them is slightly inconvenient. Hence, I found about the virtual users. <br>
At first I had massive issues configuring them. But here we go <br>
For this documentation I have been using those sources: <br>
https://wiki.archlinux.org/title/Virtual_user_mail_system_with_Postfix,_Dovecot_and_Roundcube <br>
https://doc.dovecot.org/2.3/configuration_manual/virtual_users/ <br>
https://www.postfix.org/VIRTUAL_README.html <br>
### Postfix 
1. Verify (or create) you have vmail user and group. <br>
    ```
    groupadd -g 5000 vmail
    useradd -u 5000 -g vmail -s /usr/bin/nologin -d /home/vmail -m vmail
    ```
2. Config for ``/etc/postfix/main.cf``
    ```
    #explained Milestone 1
    myhostname = mail.companyc.c
    mydomain = companyc.c
    myorigin = $mydomain
    mydestination = localhost
    inet_interfaces = all
    mynetworks = 0.0.0.0/0
    relayhost =
    
    #What domains to handle similar to myorigin
    virtual_mailbox_domains = companyc.c
   
    #Base directory of where virtual users's emails will be written to 
    virtual_mailbox_base = /var/mail/vhosts
   
    #This is location where you define virtual users and their location
    virtual_mailbox_maps = lmdb:/etc/postfix/vmailbox
   
    #No idea it's in documentation tho
    virtual_minimum_uid = 5000
   
    #uid and guid of your vmail user
    virtual_uid_maps = static:5000
    virtual_gid_maps = static:5000
   
    #This is used to define email aliases for example info@companyc.c can be pointed to hr@companyc.c it's like redirect
    virtual_alias_maps = lmdb:/etc/postfix/virtual
    ```
3. Sample of files
   - /etc/postfix/vmailbox
   ```
    usera@companyc.c companyc.c/usera/
    ```
   - /etc/postfix/virtual (is empty, but exist)
4. Generate lmdb files using ``postmap /etc/postfix/vmailbox`` and ``postmap /etc/postfix/virtual``
5. Create required directory ``/var/mail/vhosts/companyc.c/``
6. Assign correct permissions ``chown -R vmail:vmail /var/mail/vhosts/``

At this point you should be able to receive email. When received email it will create a directory (``/var/mail/vhosts/companyc.c/usera``) and you should be able to see email inside.  <br>
Dovecot needs separate configuration. So no login via email client yet.  

### Dovecot
Like mentioned above postfix and dovecot are separate software. Postfix only provides mailbox for usera@companyc.c however that is about it. <br>
We have to recreate this user in dovecot and point to correct directories <br>
1. Generate password using ``doveadm pw -s SHA512-CRYPT``
2. Create ``/etc/dovecot/users`` <br>
   Here is my sample <br>
   user@domain:password::(email user) vmail:(email group) vmail::path to user mailbox 
   ```
    usera@companyc.c:{SHA512-CRYPT}$6$mXv1/NW9ryFn/0eJ$LJMmesleTWHGkDNu4lSWYerhJuqWvio016diLuTiU8EPFdIAvnK3lRcWFkoFkch0NdNAWmO7.uxPUoMqsUMwh.::vmail:vmail::/var/mail/vhosts/companyc.c/usera
   ```
3. Under ``/etc/dovecot/conf.d/10-auth.conf`` <br>
   I commented because i don't want PAM authentication enabled
   ```#!include auth-system.conf.ext``` <br> 
   And i uncommented <br>
   ```!include auth-passwdfile.conf.ext``` 
4. Config for ``/etc/dovecot/conf.d/auth-passwdfile.conf.ext`` <br>
   I changed default_fields and the source files for the databases <br>
   ```
   passdb {
   driver = passwd-file
   args = scheme=CRYPT username_format=%u /etc/dovecot/users
   }
   userdb {
   driver = passwd-file
   args = username_format=%u /etc/dovecot/users
      default_fields = uid=vmail gid=vmail
    }
   ```
5. Additionally under ```/etc/dovecot/conf.d/10-mail.conf``` <br>
    I added line ``mail_location = maildir:/var/spool/mail/vhosts/%d/%n`` <br>
    %d - evaluates to domain e.g companyc.c <br>
    %n - evaluates to user only without domain e.g usera <br>
6. I also repeated steps from Milestone1 with TLS certificate  (``conf.d/10-ssl.conf``) <br>
   ```
   ssl_cert = </etc/ssl/certs/dovecot.pem  # Milestone1 covers that
   ssl_key = </etc/ssl/private/dovecot.pem # Milestone1 covers that
   ssl_dh = </etc/dovecot/dh.pem # Milestone1 covers that
   ```

At this point you should be able to log in. Send receive email via virtual user. <br>
![](MailServers/Milestone2/Proof1.png) <br>
![](MailServers/Milestone2/Proof2.png) <br>
![](MailServers/Milestone2/Proof3.png) <br>

## Milestone 3
This will be done when I don't know, but it will be