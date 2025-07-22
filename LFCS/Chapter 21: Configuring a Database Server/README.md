# Chapter 21: Configuring a Database Server
I set up mariadb multiple times, in fact I used it for development of https://github.com/MateuszSerafin/TBSync <br>
I will go briefly over it <br>

## Installation
1. Install package <br>
   ```
   [root@server ~]# pacman -S mariadb
   resolving dependencies...
   looking for conflicting packages...
    
   Packages (4) jemalloc-1:5.3.0-5  mariadb-clients-11.8.2-1  mariadb-libs-11.8.2-1  mariadb-11.8.2-1
    
   Total Download Size:    36.20 MiB
   Total Installed Size:  248.24 MiB
    
   :: Proceed with installation? [Y/n] y
   :: Retrieving packages...
   jemalloc-1:5.3.0-5-x86_64                                                       406.9 KiB  2.50 MiB/s 00:00 [#################################################################] 100%
   mariadb-clients-11.8.2-1-x86_64                                                1851.4 KiB  6.82 MiB/s 00:00 [#################################################################] 100%
   mariadb-libs-11.8.2-1-x86_64                                                      5.4 MiB  15.9 MiB/s 00:00 [#################################################################] 100%
   mariadb-11.8.2-1-x86_64                                                          28.6 MiB  27.5 MiB/s 00:01 [#################################################################] 100%
   Total (4/4)                                                                      36.2 MiB  34.3 MiB/s 00:01 [#################################################################] 100%
   (4/4) checking keys in keyring                                                                               [#################################################################] 100%
   (4/4) checking package integrity                                                                             [#################################################################] 100%
   (4/4) loading package files                                                                                  [#################################################################] 100%
   (4/4) checking for file conflicts                                                                            [#################################################################] 100%
   (4/4) checking available disk space                                                                          [#################################################################] 100%
   :: Processing package changes...
   (1/4) installing mariadb-libs                                                                                [#################################################################] 100%
   Optional dependencies for mariadb-libs
       krb5: for gssapi authentication [installed]
   (2/4) installing jemalloc                                                                                    [#################################################################] 100%
   Optional dependencies for jemalloc
       perl: for jeprof [installed]
   (3/4) installing mariadb-clients                                                                             [#################################################################] 100%
   (4/4) installing mariadb                                                                                     [#################################################################] 100%
   :: You need to initialize the MariaDB data directory prior to starting
      the service. This can be done with mariadb-install-db command, e.g.:
        # mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
   ```
2. Initialize MariaDB <br>
   ```
   [root@server ~]# mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
   Installing MariaDB/MySQL system tables in '/var/lib/mysql' ...
   OK
    
   ```
3. Start it <br>
   ```
   [root@server ~]# systemctl start mariadb
   [root@server ~]#
   ```
4. Finished, now you can do sql things like create users, databases or remove data without backups <br>
   ```
   [root@server ~]# mariadb
   Welcome to the MariaDB monitor.  Commands end with ; or \g.
   Your MariaDB connection id is 4
   Server version: 11.8.2-MariaDB Arch Linux
   
   Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
   
   Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
    
   MariaDB [(none)]>
   ```

## Configuring the Database Server
On arch linux the configuration files are located in ``/etc/my.cnf.d/`` <br>
My study guide covers some of the options so for the purpose of exam I will go through them <br>

| Line                             | Description                                                                                                                                                                                 |
|----------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| datadir=/var/lib/mysql           | Where datafiles are stored (databases etc)                                                                                                                                                  |
| socket=/var/lib/mysql/mysql.sock | Indicates where socket should be created                                                                                                                                                    |
| bind_address=0.0.0.0             | What network interface should be database bound to (0.0.0.0 means it listens on all interfaces)                                                                                             |
| port=20500                       | On what port should database be on                                                                                                                                                          |
| innodb_buffer_pool_size=256M     | Amount of memory to use as buffer pool for most frequent queries                                                                                                                            |
| skip_name_resolve=0              | Whether hostnames will be resolved for incoming connections (You can configure permissions based on hostname, I did not know that) Note: Disable if not necessary will speed up the queries |
| query_cache_size=0               | Caches SELECT queries, if they are exactly the same if it will return that. You specify the size of this cache using size e.g 100M, 0 means disabled                                        |
| max_connections                  | Self explanatory                                                                                                                                                                            |
| thread_cache_size=0              | Number of threads that server allocates for reuse after a client disconnects (From performance stand point it's cheaper to reuse a thread rather than to create a new one)                  |

Note: Linux sockets allow communication with other processes, in this case ``mariadb`` command is a management tool, it connects to our database via this socket additionally I am pretty sure you could remove all networking from the config and configure an app to just use this socket to communicate with database <br>
