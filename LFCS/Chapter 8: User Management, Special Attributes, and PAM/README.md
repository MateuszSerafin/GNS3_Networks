# Chapter 8: User Management, Special Attributes, and PAM
## Users Accounts and Groups
To add user account we use ``useradd`` 
```
[root@server ~]# useradd -m testuser
[root@server ~]# 
```
Note: Option `-m` means that we create a home directory which would be under ``/home/``
```
[root@server ~]# ls /home/
smbuser  testuser
```

Underneath this command some things happen: <br>
- User directory is created (unless you didn't use option `-m`)
- From ``/etc/skel/`` (which is a template for user home folder) files are copied and correct permissions are assigned
- A group is created with the same name as the user account

Some additional tasks might happen depending on your linux distribution <br>

Users are stored under ``/etc/passwd`` file <br>
```
[root@server default]# cat /etc/passwd
root:x:0:0::/root:/usr/bin/bash
...
testuser:x:1001:1001::/home/testuser:/usr/bin/bash
```
Each record for a user consists of
```
[username]:[x]:[UID]:[GID]:[Comment]:[Home directory]:[Default shell]
```

| Field            | Description                                                                                                                  |
|------------------|------------------------------------------------------------------------------------------------------------------------------|
| [username]       | Self explanatory                                                                                                             |
| [x]              | Indicates if account is protected by a shadowed password in /etc/shadow                                                      |
| [UID]            | User Identification which is an integer                                                                                      |
| [GID]            | Primary Group ID to which user belongs to which is an integer                                                                |
| [Comment]        | Self explanatory                                                                                                             |
| [Home directory] | Absolute path to home directory (users do not need to have home in /home/ directory, you can mess it up as much as you like) |
| [Default shell]  | The path of an executable that will run when user logins (technically I am correct)                                          |

Note to [Default shell]: The reason I am correct is that it literarily points to executable. You can execute /bin/bash as a current user, and you are going to open bash in bash <br>
Additionally this is used for additional security as an example <br>
```
ftp:x:14:11::/srv/ftp:/usr/bin/nologin
http:x:33:33::/srv/http:/usr/bin/nologin
nobody:x:65534:65534:Kernel Overflow User:/:/usr/bin/nologin
```
Those are special users for the purpose of for example running an http server or ftp server. They have set as a [Default shell] ``nologin`` <br>
All ``nologin`` does is prints a message and returns <br>
```
[root@server default]# nologin
This account is currently not available.
```
Which means if you try to log in to this user, you will see this message, and you will be logged off. <br>
While what I said is true additional security measures are required, if you would just do ``ssh userwithnologin@something`` it would log you off. <br>
BUT if you would do ``ssh userwithnologin@something /bin/bash`` it would allow it, and you would have normal shell access. <br>

Additionally, user passwords are stored in ``/etc/shadow``
```
[root@server default]# cat /etc/shadow
root:$y$j9T$0OMHua8jEjkWcrmx4NyQ5/$YgHhGjfcD2reFLaPlPyD4.7I.tibtcUIdHfedqEmwA3:20274::::::
bin:!*:20274::::::
daemon:!*:20274::::::
mail:!*:20274::::::
....
```
With the following format <br>
```
[username]:[password]:[last_password_change]:[min_days]:[max_days]:[warn_days]:[inactive_days]:[expire_date]:[reserved]
```

| Field                  | Description                                                                                                              |
|------------------------|--------------------------------------------------------------------------------------------------------------------------|
| [username]             | Self explanatory                                                                                                         |
| [password]             | Self explanatory, but is also hashed                                                                                     |
| [last_password_change] | Self explanatory                                                                                                         |
| [min_days]             | The minimum number of days required between password changes                                                             |
| [max_days]             | The maximum number of days the password may be used before it must be changed                                            |
| [warn_days]            | The number of days before the password expires that the user is warned                                                   |
| [inactive_days]        | The number of days after the password expires that the account remains active. After this period, the account is locked. |
| [expire_date]          | The date on which the account expires (in days since the Unix epoch)                                                     |
| [reserved]             | Reserved field for future use                                                                                            |

Note: Not setting user password means we just cannot log in to it (via ssh or console), but the user is perfectly fine to use with services, we can log to this user via sudo ``sudo -i -u`` <br>

To change user password we do ``passwd`` which will change password for current user or ``passwd (otheruser)`` to change password for other user <br>


As a continuation, groups are stored under ``/etc/group`` file <br>
```
[root@server default]# cat /etc/group
root:x:0:root
...
testuser:x:1001:
[root@server default]# 
```
The fields are as follows <br>
```
[Group name]:[Group password]:[GID]:[Group members]
```

| Field            | Description                                                 |
|------------------|-------------------------------------------------------------|
| [Group name]     | Self explanatory                                            |
| [Group password] | X there indicates that group passwords are not being used   |
| [GID]            | Group Identification, integer                               |
| [Group members]  | Comma separated list of users who are members of this group |

We can modify user with ``usermod`` command <br>

| Command                               | Description                                                    |
|---------------------------------------|----------------------------------------------------------------|
| usermod testuser -e 2025-07-14        | Sets the date when user will expire and account will be locked |
| usermod testuser -aG ftp              | Adds user to a group                                           |
| usermod testuser -rG ftp              | Removes user from a group                                      |
| usermod testuser --home /tmp          | Changes home directory for a user                              |
| usermod testuser --shell /bin/nologin | Changes default shell for a user                               |
| usermod testuser -L                   | Lock user account                                              |
| usermod testuser -L                   | Unlock user account                                            |

Note: You can do it also by editing ``/etc/passwd`` or ``/etc/shadow`` <br>

We can create groups with command ``groupadd`` 
```
[root@server default]# groupadd thisisatestgroup
[root@server default]# 
```
And delete them with ``groupdel`` 
```
[root@server default]# groupdel thisisatestgroup
[root@server default]# 
```
To delete user account with home directory we do <br>
```
[root@server default]# userdel -r testuser
[root@server default]# 
```

Note: If you delete the account the UID, is again available. This means that if previous user was owner of XYZ directory and new user hits the same UID new user will have access to those files because access to files is determined by UID, GUID <br>
This can pose a security risk, as the new user would gain access to all files previously owned by the old user. For this reason, some systems avoid reusing UIDs.

## Special Permissions
### SETUID
When this permission is set. The program inherits the effective privileges of the program's owner <br>
Which is a security concert and the amount of those executables should be limited to minimum <br>
However for programs as ``passwd`` something like that must exist. <br>
In short this is a limited privilege escalation <br>

To check if executable has this flag set we can use following command 
```
[root@server bin]# ls -l passwd 
-rwsr-xr-x 1 root root 80856 Jun 27 07:35 passwd
```
And we can clearly see the flag ``s`` is in there <br>

To add the flag we can use ``chmod u+s file`` and to remove we can use ``chmod u-s file``
### SETGID
Similarly to SETUID, when the program is executed it runs with the privileges of the file's group owner <br>
Additionally it makes directories and files inherit the group ownership of the directory <br>

To add the flag we can use ``chmod g+s file/directory`` and to remove we can use ``chmod g-s file/directory``

### STICKY BIT
Let's take as an example ``/tmp`` directory <br>
```
drwxrwxrwt   8 root root   160 Jul 14 12:31 tmp
```
It has the sticky bit set which is shown by ``t`` flag <br>

Now let's look at permissions, the permissions are 777. It means everyone can create, delete, execute files. However, in this particular case we might not want any user to delete other user's tmp data <br>
Hence setting sticky bit, will mean only owner of a file or directory can delete it, even the folder itself has 777 permissions. 

Stick bit is only relevant to folders <br>
To add the flag we can use ``chmod o+t directory`` and to remove we can use ``chmod o-t directory``
Alternatively with decimal format ``chmod 1XXX `` to add the flag or ``chmod XXX`` to remove it 

### Special File Attributes

| Command         | Description                                        |
|-----------------|----------------------------------------------------|
| chattr +i file1 | File cannot be moved, renamed, modified or deleted |
| chattr +a file2 | Append-Only mode aka content can only be added     |

## Accessing the Root Account and Using sudo
Without sudo, we can use command ``su`` after providing root password we will log in as root <br>
```
[testuser@server ~]$ su
Password: 
[root@server testuser]# 
```
This requires user to know the password of root which is not ideal <br>
Alternatively we can use sudo which needs to be configured <br>
The config for sudo is under ``/etc/sudoers`` <br>
Let's start with line 
```
Defaults secure_path="/usr/local/sbin:/usr/local/bin:/usr/bin"
```
Which allows you to specify directories that will be used for sudo, it prevents using user-specified directories which could harm the system <br>
The actual config that you have to do is add the user which will be allowed to user sudo command <br>
You have multiple options (damn again table)

| Line                                  | Description                                                              |
|---------------------------------------|--------------------------------------------------------------------------|
| testuser ALL=/bin/pacman -Sy          | Allows testuser to only sudo ``pacman -Sy`` command                      |
| testuser ALL=(ALL:ALL) ALL            | Allows testuser to execute all commands under sudo                       |
| testuser ALL=(ALL:ALL) NOPASSWD: ALL  | Allows testuser to execute all commands under sudo without user password |

Note: You can do something like that to allow multiple commands without giving access to everything
```
Cmnd_Alias VIEW = /bin/cat /var/log/messages, /bin/head /var/log/messages, /bin/tail /var/log/messages, /bin/tailf /var/log/messages
testuser ALL=NOPASSWD: VIEW
```
Also, you can specify the group instead of the user, so you could create a group admins and manage permissions this way <br>
```
%admins ALL=/bin/pacman -Sy 
```

## PAM (Pluggable Authentication Modules)
PAM is a basically a system authenticator, it standardizes a way to authenticate the users and allows developers to use it. Which means that if developer is writing some software for Linux, developer doesn't need to handle authentication by him self resulting in more secure software <br>
Additionally it allows for funky stuff such as authentication with Active Directory domain when configured with additional packages <br>
PAM is used by common services such as SSHD, it handles console logins and is integrated into the system itself. 

There are 4 types of PAM modules

| Module   | Description                                                                   |
|----------|-------------------------------------------------------------------------------|
| auth     | Checks if user provided valid credentials                                     |
| account  | Grants privileges and verifies users                                          |
| password | Password management (updates, complexity etc)                                 |
| session  | Indicates what should be done before and/or after the authentication succeeds |

Additionally, there are "controls"

| Control    | Description                                                                                                   |
|------------|---------------------------------------------------------------------------------------------------------------|
| required   | The module must succeed otherwise authentication is failed but module is still processed                      |
| requisite  | The module must succeed otherwise authentication is immediately failed and no further processing is happening |
| sufficient | If this module succeeds, no future processing is required the access is granted                               |
| optional   | The module is not required to succeed (it's optional)                                                         |

So actually looking at the ``/etc/pam.d/system-auth`` <br>
```
auth       required                    pam_faillock.so      preauth
# Optionally use requisite above if you do not want to prompt for the password
# on locked accounts.
-auth      [success=2 default=ignore]  pam_systemd_home.so
auth       [success=1 default=bad]     pam_unix.so          try_first_pass nullok
auth       [default=die]               pam_faillock.so      authfail
auth       optional                    pam_permit.so
auth       required                    pam_env.so
auth       required                    pam_faillock.so      authsucc
# If you drop the above call to pam_faillock.so the lock will be done also
# on non-consecutive authentication failures.

-account   [success=1 default=ignore]  pam_systemd_home.so
account    required                    pam_unix.so
account    optional                    pam_permit.so
account    required                    pam_time.so

-password  [success=1 default=ignore]  pam_systemd_home.so
password   required                    pam_unix.so          try_first_pass nullok shadow
password   optional                    pam_permit.so

-session   optional                    pam_systemd_home.so
session    required                    pam_limits.so
session    required                    pam_unix.so
session    optional                    pam_permit.so
```

In this file we can see that first authentication checks are performed by different modules <br>
Then account management checks are performed followed by checking user passwords (if it's expired, if it requires to be changed) <br>
Lastly user session is established <br>

Note: PAM is pretty low level, I used it twice and both cases to add Active Directory to the Linux machine. My study guide shows just some basics I believe on exam they just want you to know it exists and what you can do with it <br>
Note2: Going through every single .so module while it would be cool, it's slight overkill, you can easily google it https://linux.die.net/man/8/pam_faillock <br> 
Additionally with pam you can configure 2-factor authentication, OTP and other security enhancements