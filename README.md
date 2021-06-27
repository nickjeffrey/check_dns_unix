# check_dns_unix
nagios check to verify DNS configuration for UNIX-like operating systems

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
