# check_dns_unix
nagios check to verify DNS configuration for UNIX-like operating systems.  The following items are checked:
   - confirm the current machine has forward and reverse name resolution (A and PTR records in DNS)
   - confirm PTR record returned by DNS match an IP address bound to a local ethernet interface
   - confirm the local IP address is not 0.0.0.0
   - confirm that localhost resolves to 127.0.0.1
   - confirm default DNS domain suffix is defined
   - ping each defined DNS server
   


# Requirements
perl, ssh  on nagios server

# Configuration

This script is executed remotely on a monitored system by the NRPE or check_by_ssh  methods available in nagios.

If you hare using the check_by_ssh method, you will need a section in the services.cfg file on the nagios server that looks similar to the following.
This assumes that you already have ssh key pairs configured.
```
    define service {
       use                             generic-8x5-service
       hostgroup_name                  all_aix,all_linux
       service_description             DNS
       check_command                   check_by_ssh!/usr/local/nagios/libexec/check_dns_unix
       }
```

Alternatively, if you are using the NRPE method, you should have a section similar to the following in the services.cfg file:
```
    define service{
       use                             generic-8x5-service
       hostgroup_name                  all_aix,all_linux
       service_description             DNS
       check_command                   check_nrpe!check_dns_unix -t 30
       normal_check_interval           240     ; only check every 4 hours
       notification_options            c,r     ; Send notifications about critical and recovery events
       }
```

If you are using the NRPE method, you will also need a command definition similar to the following on each monitored host in the /usr/local/nagios/nrpe/nrpe.cfg file:
```
    command[check_dns_unix]=/usr/local/nagios/libexec/check_dns_unix
```

# Output
Command output will look similar to one of the following:
```
DNS OK hostname=myserver.example.com ipaddr=192.168.99.19 localhost=127.0.0.1 resolver=bind nameservers=192.168.99.3 192.168.99.4
```
```
DNS WARN - Missing domain and search parameters from /etc/resolv.conf.  Please add an entry similar to one or both of the following:  domain example.com  search example.com
```
```
DNS WARN - Could not find an A record for MyServer in DNS.  Forward DNS lookups for MyServer are failing against the 192.168.99.3 DNS server.  Please ask the DNS administrator to add an A record for MyServer on the 192.168.99.3 DNS server.
```
```
DNS WARN - PTR record for 192.168.99.143 is pointing at hostname MyServer, but the IP address 192.168.99.143 is not bound to any network interface on MyServer. Please fix up the A and PTR records for hostname MyServer on DNS server 192.168.99.3, or add IP address 192.168.99.143 to a network interface on MyServer.
```
