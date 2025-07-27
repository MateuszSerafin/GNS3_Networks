# Chapter 24: The Firewall
## The Basics About iptables
IPTables can take following actions to packets <br>
- Accept (Self Explanatory)
- Reject (Drops the packet and sends message about it to source)
- Drop (Drops the packet without informative message)
- Forward (Forwards the packet to other network)

There are 3 chains where we can apply actions <br>

| Chain      | Description                                                                     |
|------------|---------------------------------------------------------------------------------|
| INPUT      | Handles packets incoming to our system and are destined to our local programs   |
| OUTPUT     | Handles packets originating from our local network which are to be send outside |
| FORWARD    | Handles packet forwarding (not destination of our network)                      |

We can use ``iptables -L`` to list current policies <br>
```
[root@server ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
[root@server ~]# 
```

## Adding Rules
To add a rule to firewall we can use following command <br>
```
iptables -A chain_name criteria -j target
```
Where 
- -A stands for Append
- chain_name is either INPUT, OUTPUT or FORWARD
- target is the action (ACCEPT, REJECT or DROP)
- criteria is the set of conditions against which the packets are to be examined 

Note: There is A LOT of options for criteria e.g <br>
- Match by protocol (TCP, UDP, ICMP)
- Match by source/destination port 
- Match by source/destination ip
- Match by tcp state (NEW, ESTABLISHED, RELATED, INVALID)
- Match by incoming/outgoing interface 

...

Let's create a simple rule that will drop ICMP requests <br>
Before I start I can ping the virtual machine without issues <br>
```
[mateusz@MateuszLaptop ~]$ ping 10.21.37.15
PING 10.21.37.15 (10.21.37.15) 56(84) bytes of data.
64 bytes from 10.21.37.15: icmp_seq=1 ttl=62 time=1.40 ms
64 bytes from 10.21.37.15: icmp_seq=2 ttl=62 time=1.03 ms
```
After applying the rule <br>
```
[root@server ~]# iptables -A INPUT --protocol icmp --in-interface ens3 -j DROP
[root@server ~]# 
```
It doesn't work (as expected) <br>
```
[mateusz@MateuszLaptop ~]$ ping 10.21.37.15
PING 10.21.37.15 (10.21.37.15) 56(84) bytes of data.
^C
--- 10.21.37.15 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2082ms

[mateusz@MateuszLaptop ~]$ 
```

Using REJECT, instead of DROP looks like that <br>
```
[mateusz@MateuszLaptop ~]$ ping 10.21.37.15
PING 10.21.37.15 (10.21.37.15) 56(84) bytes of data.
From 10.21.37.15 icmp_seq=1 Destination Port Unreachable
From 10.21.37.15 icmp_seq=2 Destination Port Unreachable
From 10.21.37.15 icmp_seq=3 Destination Port Unreachable
From 10.21.37.15 icmp_seq=4 Destination Port Unreachable
^C
```

Note: You can use ``iptables -F`` to flush ALL rules you have on your firewall <br>
Alternatively you can use ``iptables -F INPUT/OUTPUT/FORWARD`` to flush specific chains  <br>

```
[root@server ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
REJECT     icmp --  anywhere             anywhere             reject-with icmp-port-unreachable

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
[root@server ~]# iptables -D INPUT --protocol icmp --in-interface ens3 -j REJECT
[root@server ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
[root@server ~]# 
```
Alternatively you can remove it by line <br>
```
[root@server ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
REJECT     icmp --  anywhere             anywhere             reject-with icmp-port-unreachable

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
[root@server ~]# iptables -D INPUT 1
[root@server ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
[root@server ~]# 
```

As an additional example we can block OUTGOING ssh connections <br>
```
[root@server ~]# iptables -A OUTPUT --protocol tcp --destination-port 22 -j DROP
[root@server ~]# ssh root@10.21.37.1
^C
[root@server ~]# ssh root@10.21.37.1
^C
[root@server ~]# iptables -F
[root@server ~]# ssh root@10.21.37.1
The authenticity of host '10.21.37.1 (10.21.37.1)' can't be established.
ED25519 key fingerprint is SHA256:3RCU6YosacTLDyMXN9nAhAf6BoUrYAh2bDpoGU4hPbU.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? ^C
[root@server ~]# 
```

## Inserting, Appending, and Deleting Rules
Let's assume we have the following chain <br>
```
Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
DROP       tcp  --  anywhere             anywhere             tcp dpt:ssh
```
If I want to permit host ``10.21.37.1`` with <br>
```
[root@server ~]# iptables -A OUTPUT --protocol tcp --destination-port 22 --destination 10.21.37.1 -j ALLOW
```
I will not work because our rule will be added at bottom and rule blocking the connections is above it <br>
```
Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
DROP       tcp  --  anywhere             anywhere             tcp dpt:ssh
ACCEPT     tcp  --  anywhere             ArchVisor            tcp dpt:ssh
```
In this case we can ``-I`` which stands for Insert, where ``-A`` stands for append, Append is appending at bottom, Insert is inserting at index it makes sense <br>
```
[root@server ~]# iptables -I OUTPUT 1 --protocol tcp --destination-port 22 --destination 10.21.37.1 -j ACCEPT
```
And we can see that our ``ACCEPT`` rule is first <br>
```
Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     tcp  --  anywhere             ArchVisor            tcp dpt:ssh
DROP       tcp  --  anywhere             anywhere             tcp dpt:ssh
```
Cool <br>

To delete a rule we can use ``-D`` option which works similar to ``-I`` option <br>
```
[root@server ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     tcp  --  anywhere             ArchVisor            tcp dpt:ssh
DROP       tcp  --  anywhere             anywhere             tcp dpt:ssh
[root@server ~]# iptables -D OUTPUT 2
[root@server ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     tcp  --  anywhere             ArchVisor            tcp dpt:ssh
```
Note: We can use ``iptables -nL -v --line-numbers`` to list our rules in more verbose way which shows indexes of rules, and we can delete rules exactly knowing the index we want to remove <br>

We can replace rules with ``-R`` option <br>
```
[root@server ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     tcp  --  anywhere             ArchVisor            tcp dpt:ssh
[root@server ~]# iptables -R OUTPUT 1 --protocol tcp --destination-port 22 --destination 10.21.37.99 -j ACCEPT
[root@server ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     tcp  --  anywhere             10.21.37.99          tcp dpt:ssh
```

## NAT
NAT example from http://wiki.archlinux.org/title/Internet_sharing
```
iptables -t nat -A POSTROUTING -o internet0 -j MASQUERADE
iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i net0 -o internet0 -j ACCEPT
```
This will create a NAT, OUT interface is ``internet0`` <br>
incoming interface is ``net0`` (more interfaces can be added) <br>

## Saving rules 
Iptables rules are not persistent meaning if you reboot it's all gone <br>
What you want to do is dump the rules with ``iptables-save`` <br>
```
[root@server ~]# iptables-save 
# Generated by iptables-save v1.8.11 on Thu Jul 24 20:20:06 2025
*filter
:INPUT ACCEPT [22841:1991494]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [22845:2031226]
-A OUTPUT -d 10.21.37.99/32 -p tcp -m tcp --dport 22 -j ACCEPT
COMMIT
# Completed on Thu Jul 24 20:20:06 2025
```
You then can paste it into ``/etc/iptables/iptables.rules `` <br>
If you enable the service <br>
```
[root@server ~]# systemctl enable iptables
Created symlink '/etc/systemd/system/multi-user.target.wants/iptables.service' → '/usr/lib/systemd/system/iptables.service'.
```
It will read the rules from this file on boot making your rules persistent <br>
Note: You can load rules manually without service with command <br>
```
[root@server ~]# iptables-restore < /etc/iptables/iptables.rules
```
Note: Making persistent saves differs on distribution

## Summary
Study guide just covers basics and even mentions it, iptables is not only a firewall but also manages routing (partially) it's really flexible and there are alternatives such as ``nftables`` <br>