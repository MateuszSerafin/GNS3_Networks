# Chapter 20: Setting up a Caching DNS Server
I covered pretty much everything in this chapter under ``LinuxSandbox/MailServers`` <br>
I configured my own DNS to handle resolving for mail servers <br>
The only difference between what I did and this is, This chapter would just do <br>
```
options {
.
listen-on port 53 { 127.0.0.1; 192.168.0.18};
allow-query { localhost; 192.168.0.0/24; };
recursion yes;
forwarders {
8.8.8.8;
8.8.4.4;
};
…
}
```
And if you would point your computers to this dns server it would reach to 8.8.8.8 and cache the results <br>

The only thing that is worth mentioning is <br>
```
logging {
channel query_log {
file "/var/log/bind9/query.log";
severity dynamic;
print-category yes;
print-severity yes;
print-time yes;
};
category queries { query_log; };
};
```

Which would make the dns requests append in a log  <br>
Note: I configured zones, reverse zones and records in ``LinuxSandbox/MailServers`` which was already mentioned 