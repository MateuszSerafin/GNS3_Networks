# Chapter 25: Static and Dynamic Routing
Let's start with a table <br>

| Command  | Description                                                       |
|----------|-------------------------------------------------------------------|
| ip link  | Network Device management (Make it up, down, add dot1x interface) |
| ip addr  | Management of ipv4/6                                              |
| ip route | Routing                                                           |
| ip rule  | Sets priority of a route                                          |

Note: ip commands are not persistent, after reboot nothing is saved. Nonetheless, knowing those tools is useful <br>
Additionally configuring network interfaces would be usually done via Systemd, NetworkManager or other package depending on your distribution <br>
Additionally try to not match different packages for network management (Such as systemd and NetworkManager) on one system <br>

## ip link

We can use command ``ip link show`` to show current network interfaces and their statuses <br>
```
[root@server ~]# ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 0c:c7:79:40:00:00 brd ff:ff:ff:ff:ff:ff
    altname enp0s3
    altname enx0cc779400000
```

We can use command ``ip link set interface {up | down}`` to well bring the interface up or down <br>
```
[root@server ~]# ip link set ens3 down
```

Note: We can do 
```
ip link add link eth0 name eth0.5 type vlan id 5
```
Which will create a subinterface eth0.5 with vlan 5, this command was really handy for setting up PacketFence <br>

## ip route
We can use ``ip route`` command to manage network routes if you do not put any arguments, it will display routing table <br>
```
[root@server ~]# ip route
default via 10.21.37.1 dev ens3 proto dhcp src 10.21.37.15 metric 100 
8.8.8.8 via 10.21.37.1 dev ens3 proto dhcp src 10.21.37.15 metric 100 
10.21.37.0/24 dev ens3 proto kernel scope link src 10.21.37.15 metric 100 
10.21.37.1 dev ens3 proto dhcp scope link src 10.21.37.15 metric 100 
```
Note: There are more commands to display routing table but this is the standard way <br>

To add a route we can do <br>
```
ip route add 10.0.0.0/24 via 192.168.0.15 dev ens3
```
Or to add default route we can do <br>
```
ip route add default via 10.21.37.1 dev ens3
```
Providing ``dev ens3`` is not required as Linux will manage to find the interface with correct subnet and forward the packets, but it's just good practise <br>

To remove the route we do the same command as with addition of the routes, but we specify ``del`` instead of ``add`` <br>
```
ip route del 10.0.0.0/24 via 192.168.0.15 dev ens3
```