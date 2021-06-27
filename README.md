# check_aix_misc

Nagios plugin for performing miscellaneous checks of AIX systems
```
    -confirm sudo is installed
    - sanity checks on boot logical volume
    - confirm /etc/rc.tcpip is executable
    - confirm loopback entry exists in /etc/hosts (required for SNMP daemons) 
    - confirm aio0 device exists (AIX 5.3 only)
    - confirm that no user has more than 1000 mail messages (need to elevate privs with sudo)
    - confirm /etc/resolv.conf contains valid info
    - confirm we can ping name servers
    - confirm /var/adm/messages exists for syslog
    - syslog sanity checks
    - confirm we only have one default route
    - ping default router
    - confirm hostname does not contain FQDN
    - confirm sendmail queue is not filling up with queued messages
    - confirm host exists in DNS
    - if an SMTP smart host is listed in /etc/mail/sendmail.cf, confirm we can ping it
    - check for important processes
    - syslog sanity checks
    - confirm each hdisk in rootvg has a boot logical volume (ie is bootable)
    - confirm boot logical volume exists
    - confirm all varied on volume groups are readable (checks for corrupt volume groups)
    - check for problems with software RAID mirrors, stale PP, missing PV
    - check system attention light, RMC connection to HMC
    - check /etc/rc.tcpip permissions
    - confirm /etc/rc.local is executed at boot time
    - confirm recent TSM backup (if TSM client is installed)
    - confirm recent Symantec NetBackup activity (if installed)
    - confirm recent Avamar backup activity (if installed)
    - confirm system attention light is not illuminated
    - confirm RMC daemon is communicating with HMC
    - NIM server/client sanity checks
    - sanity checks for system dump device
    - check for recent reboot (within last hour)
    - check for iocp device
    - confirm /var/adm/wtmp file exists
    - confirm NFS source ports are set correctly
    - validate backup clients for IBM TSM, EMC Avamar, Symantec NetBackup
```



# Requirements
ksh, ssh, sudo on nagios server

# Configuration

This script is executed remotely on a monitored system by the NRPE or check_by_ssh methods available in nagios.  

If you are using the check_by_ssh method, you will need to add a section similar to the following to the services.cfg file on the nagios server.  
This assumes you already have ssh key pairs configured.
```
    define service {
       use                             generic-service
       hostgroup_name                  all_aix
       service_description             AIX misc checks
       check_command                   check_by_ssh!/usr/local/nagios/libexec/check_aix_misc
       }
```

Alternatively, if you are using the check_nrpe method, you will need to add a section similar to the following to the services.cfg file on the nagios server.  
This assumes you already have ssh key pairs configured.
```
   define service{
      use                             generic-service
      hostgroup_name                  all_aix
      service_description             AIX misc checks
      check_command                   check_nrpe!check_aix_misc -t 30
      notification_options            c,r                     ; Send notifications about critical and recovery events
      }
```

If you are using the NRPE method, you will also need a command definition similar to the following on each monitored host in the /usr/local/nagios/nrpe/nrpe.cfg file:
```
    command[check_aix_misc]=/usr/local/nagios/libexec/check_aix_misc
```


This script executes as the nagios user, but there are a few steps that need to be executed with root privileges.
We handle this by using sudo to execute certain commands, so you will need to ensure that sudo is installed, and entries similar to the following exist in the /etc/sudoers file:
```
    User_Alias      NAGIOS_USER = nagios
    Cmnd_Alias      LS = /usr/sbin/ls
    Cmnd_Alias      GREP = /usr/bin/grep
    Cmnd_Alias      BOOTLIST = /usr/bin/bootlist
    Cmnd_Alias      NIMCLIENT = /usr/sbin/nimclient
    Cmnd_Alias      SYSDUMPDEV = /usr/bin/sysdumpdev
    Cmnd_Alias      STARTSRC = /usr/bin/startsrc
    NAGIOS_USER ALL = (root) NOPASSWD: LS,GREP,BOOTLIST,NIMCLIENT,SYSDUMPDEV,STARTSRC
```


