---
layout: post
title: Living with enabled SELinux
tags:
    - SELinux
author: Radosław Śmigielski
---

Why?
====
How many times things didn't work for you and tou end up running one of these:
```
# setenforce 0
# setenforce Permissive
# echo 0 > /selinux/enforce
```
Below post hopefully will show you better way of living with SELinux enabed.


Intro
=====
Let's start from checking current status of SELinux with _**sestatus**_
```
[radek@photon][/tmp]$ sestatus 
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Memory protection checking:     actual (secure)
Max kernel policy version:      31
```
Line _**SELinux status**_ shows you current SELinux status.

On most of the system there is _**auditd**_ daemon running which manages
SELinux policies but also logs SELinux events to log file, on Redhat based
system this is the log file: _/var/log/audit/audit.log_

Example
=======
Classic example of SELinux causing some problems looks like this:
```
[root@localhost cbis]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since Sat 2020-03-21 05:15:25 UTC; 7s ago
  Process: 46754 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=1/FAILURE)
  Process: 46751 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)

Mar 21 05:15:25 localhost.localdomain systemd[1]: Starting The nginx HTTP and reverse proxy server...
Mar 21 05:15:25 localhost.localdomain nginx[46754]: nginx: [alert] could not open error log file: open() "/var/log/nginx/error.log" failed (13: Permission denied)
Mar 21 05:15:25 localhost.localdomain nginx[46754]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Mar 21 05:15:25 localhost.localdomain nginx[46754]: 2020/03/21 05:15:25 [emerg] 46754#0: open() "/var/log/nginx/access.log" failed (13: Permission denied)
Mar 21 05:15:25 localhost.localdomain nginx[46754]: nginx: configuration file /etc/nginx/nginx.conf test failed
Mar 21 05:15:25 localhost.localdomain systemd[1]: nginx.service: control process exited, code=exited status=1
Mar 21 05:15:25 localhost.localdomain systemd[1]: Failed to start The nginx HTTP and reverse proxy server.
Mar 21 05:15:25 localhost.localdomain systemd[1]: Unit nginx.service entered failed state.
Mar 21 05:15:25 localhost.localdomain systemd[1]: nginx.service failed.
```
Usually there is no meaningful error in service logs file, all permissions
are set in proper way but you keep getting "_Permission denied_" or similar
confusing error. And finally magic command "_setenforce Permissive_" resolves your problem.

But what if you can't run your system in _Permissive_ mode?


How to resolve SELinux denials
==============================

How to identify problem?
------------------------
First examine _/var/log/audit/audit.log_ file and try to identify lines
related to your problem what is not always obvious.
```
grep denied /var/log/audit/audit.log
```

If you do not see any relevant denials in audit.log file, it could be that your
system runs with _**dontaudit**_ rule enabled, which is possible reason
of silent denials. This is how you can disable this rule:
```
semanage dontaudit off
```

And finally tiny example, Jekyll is a static site generator running inside
container failling to read files.
```
type=AVC msg=audit(1584772223.906:5543): avc:  denied  { read } for  pid=776207 comm="server.rb:286" name="404.html" dev="sda3" ino=580053375 scontext=system_u:system_r:container_t:s0:c717,c753 tcontext=unconfined_u:object_r:user_home_t:s0 tclass=file permissive=0
type=AVC msg=audit(1584772735.389:5549): avc:  denied  { read } for  pid=776207 comm="server.rb:286" name="404.html" dev="sda3" ino=580053375 scontext=system_u:system_r:container_t:s0:c717,c753 tcontext=unconfined_u:object_r:user_home_t:s0 tclass=file permissive=0
type=AVC msg=audit(1584773463.635:5574): avc:  denied  { noatsecure } for  pid=778375 comm="nm-dispatcher" scontext=system_u:system_r:NetworkManager_t:s0 tcontext=system_u:system_r:initrc_t:s0 tclass=process permissive=0
type=AVC msg=audit(1584773463.635:5575): avc:  denied  { rlimitinh } for  pid=778375 comm="04-iscsi" scontext=system_u:system_r:NetworkManager_t:s0 tcontext=system_u:system_r:initrc_t:s0 tclass=process permissive=0
type=AVC msg=audit(1584773463.635:5576): avc:  denied  { siginh } for  pid=778375 comm="04-iscsi" scontext=system_u:system_r:NetworkManager_t:s0 tcontext=system_u:system_r:initrc_t:s0 tclass=process permissive=0
type=AVC msg=audit(1584773463.649:5577): avc:  denied  { noatsecure } for  pid=778377 comm="nm-dispatcher" scontext=system_u:system_r:NetworkManager_t:s0 tcontext=system_u:system_r:initrc_t:s0 tclass=process permissive=0
type=AVC msg=audit(1584773587.513:5590): avc:  denied  { getattr } for  pid=776207 comm="thread_pool.rb*" path="/srv/jekyll/_posts/2020-03-21-SELinux_living_with.md" dev="sda3" ino=55976446 scontext=system_u:system_r:container_t:s0:c717,c753 tcontext=unconfined_u:object_r:user_home_t:s0 tclass=file permissive=0
```

Identify problem
----------------
SELinux audit.log includes _**msg=audit**_ value in each and every line,
_**msg=audit**_ value let you _decrypt_ reason of access denials.

We have below line in _**/var/log/audit/audit.log**_
```
type=AVC msg=audit(1584773587.513:5590): avc:  denied  { getattr } for  pid=776207 comm="thread_pool.rb*" path="/srv/jekyll/_posts/2020-03-21-SELinux_living_with.md" dev="sda3" ino=55976446 scontext=system_u:system_r:container_t:s0:c717,c753 tcontext=unconfined_u:object_r:user_home_t:s0 tclass=file permissive=0
```
With message audit id: _1584773587.513:5590_. Let's try to translate that
message into something human readable.
```
[root@photon audit]# grep 1584773587.513:5590 /var/log/audit/audit.log | audit2why
type=AVC msg=audit(1584773587.513:5590): avc:  denied  { getattr } for  pid=776207 comm="thread_pool.rb*" path="/srv/jekyll/_posts/2020-03-21-SELinux_living_with.md" dev="sda3" ino=55976446 scontext=system_u:system_r:container_t:s0:c717,c753 tcontext=unconfined_u:object_r:user_home_t:s0 tclass=file permissive=0

        Was caused by:
                Missing type enforcement (TE) allow rule.
                You can use audit2allow to generate a loadable module to allow this access.
```

Now when we identify the problem let's try to craft some new SELinux rules for what we need.
Here is different example how to generate rules for some Ruby web applicatin
```
[root@photon audit]# grep ruby /var/log/audit/audit.log | audit2allow -m ryby_app

module ryby_app 1.0;

require {
        type user_tmp_t;
        type container_t;
        type container_runtime_t;
        type user_home_t;
        class dir { add_name create remove_name write };
        class file { create execute ioctl map open setattr unlink write };
}

#============= container_t ==============
allow container_t container_runtime_t:dir { add_name create remove_name write };
allow container_t container_runtime_t:file { create setattr unlink write };

#!!!! This avc is allowed in the current policy
allow container_t user_home_t:dir { add_name create remove_name write };

#!!!! This avc is allowed in the current policy
#!!!! This av rule may have been overridden by an extended permission av rule
allow container_t user_home_t:file { create ioctl open setattr unlink write };
allow container_t user_tmp_t:dir remove_name;

#!!!! This avc can be allowed using the boolean 'domain_can_mmap_files'
allow container_t user_tmp_t:file map;
allow container_t user_tmp_t:file { execute unlink };
```

Instead of printing the new rules on screen we can generate set of new rules and save them to file:
```
sudo grep ruby /var/log/audit/audit.log | audit2allow -m ryby_app
```
This will geberate two new files in current working directory:
```
[root@photon]# file ryby_app.pp
ryby_app.pp: SE Linux modular policy version 1, 1 sections, mod version 19, MLS, module name ryby_app\003
[root@photon]# file ryby_app.te
ryby_app.te: C++ source, ASCII text
```
At this point we can load the new set of rules:
```
sudo semodule -i ryby_app.pp
```
Above command will install new SELinux module what can be verified with **dmesg** output.

More info about SELinux [here](https://selinuxproject.org/).
