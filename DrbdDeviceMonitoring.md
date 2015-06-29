<h1> Introduction </h1>
This document explains how to monitor DRBD devices.

<h2>Table of contents</h2>

# Install #

## On all ganeti nodes ##
just install check\_drbd (http://www.monitoringexchange.org/attachment/download/Check-Plugins/Operating-Systems/Linux/check_drbd/check_drbd) somewhere in the path.


By default, this script will return CRITICAL if a device is unconfigured, and UNKNOWN if a device does not exists.
These situations can happen with ganeti when :

> - a vm is down

> - a vm is down and a node has been rebooted (then the drbd device disappears).

You may want to patch check\_drbd like this to avoid "false" alerts :
```
--- check_drbd  2010-01-26 09:10:16.000000000 +0100
+++ /usr/local/bin/check_drbd   2010-01-26 09:06:23.000000000 +0100
@@ -38,7 +38,7 @@
              'WFConnection' => { 'value' => 'CRITICAL', 'type' => 'cs' },
               'WFReportParams' => { 'value' => 'CRITICAL', 'type' => 'cs' },
              'Connected' => { 'value' => 'OK', 'type' => 'cs' },
-             'Unconfigured' => { 'value' => 'CRITICAL', 'type' => 'cs' },
+             'Unconfigured' => { 'value' => 'OK', 'type' => 'cs' },
              # DRBD 0.6
              'SyncingAll' => { 'value' => 'WARNING', 'type' => 'cs' },
               'SyncingQuick' => { 'value' => 'WARNING', 'type' => 'cs' },
@@ -261,7 +261,7 @@
        }
        foreach my $device (@devices) {
                if (!(defined($cs{$device}))) {
-                       &myexit('UNKNOWN',"Could not find device $device");
+                       &myexit('OK',"Could not find device $device");
                }
                $check{$device} = 1;
        }
```

## On ganeti master ##
put this script somewhere in the path (/usr/local/bin/drbd\_gen\_nagios.sh for ex):
```
#!/bin/bash 
# 
# Copyright (C) 2009 Maxence Dunnewind <maxence@dunnewind.net> 
# 
# This program is free software: you can redistribute it and/or modify 
# it under the terms of the GNU General Public License as published by 
# the Free Software Foundation, either version 3 of the License, or 
# (at your option) any later version. 
# 
# This program is distributed in the hope that it will be useful, 
# but WITHOUT ANY WARRANTY; without even the implied warranty of 
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the 
# GNU General Public License for more details. 
# 
# You should have received a copy of the GNU General Public License 
# along with this program.  If not, see <http://www.gnu.org/licenses/>. 

gnt-instance list  --no-header --separator=: -o name|while read vm;do 
        TMP_FILE=$(mktemp) 
        gnt-instance info -s $vm > $TMP_FILE 
        NODE_A=$(grep "nodeA" $TMP_FILE|grep -o "[^ ]\+,"|head -1|sed 
's/,//') 
        NODE_B=$(grep "nodeB" $TMP_FILE|grep -o "[^ ]\+,"|head -1|sed 
's/,//') 
        DEV_A=$(grep "nodeA" $TMP_FILE|cut -d "=" -f 2) 
        DEV_B=$(grep "nodeB" $TMP_FILE|cut -d "=" -f 2) 
        rm -rf $TMP_FILE 
        for disk in $DEV_A;do 
                echo "define service {" 
                echo " host_name                $NODE_A" 
                echo " service_description      DRBD instance $vm / 
device n° $disk" 
                echo " check_command            check_drbd!$disk" 
                echo " use                      generic-service" 
                echo "}" 
                echo "" 
        done 
        for disk in $DEV_B;do 
                echo "define service {" 
                echo " host_name                $NODE_B" 
                echo " service_description      DRBD instance $vm / 
device n° $disk" 
                echo " check_command            check_drbd!$disk" 
                echo " use                      generic-service" 
                echo "}" 
                echo "" 
        done 
done 
```

**Important** : This script only define drbd checks, so you already need
to have the hosts configured in your nagios.

## On nagios ##
Define check\_drbd, which will use check\_by\_ssh to call check\_drbd
script on the nodes.
```
define command{ 
        command_name    check_drbd 
        command_line    /usr/lib/nagios/plugins/check_by_ssh -H $HOSTADDRESS$ -l root -C "/usr/local/bin/check_drbd -d $ARG1$" 
} 
```

for example in /etc/nagiosX/conf.d/command.cfg .

Also, be sure nagios user can connect using ssh without password. If you don't have defined that, you need to :
```
# su - nagios
# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/var/run/nagios3/.ssh/id_rsa): /etc/nagios3/id_rsa
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
```

Don't forget to replace the file. If you keep /var/run, your key will be deleted after next reboot :) Also, keep the passphrase empty. Then, copy the id\_rsa.pub under /root/.ssh/authorized\_keys on all your hosts. You can also add a "from" field and a "command" one to limit access :
```
from="1.1.1.1",command="/usr/local/bin/check_drbd" ssh-rsa AAAAB3NzaC1yc2EAAAABIwA...
```

# Use #
To generate the config, after a new instance has been added / removed, or after a replace-disks, run :
```
drbd_gen_nagios.sh > drbd.cfg 
```

on ganeti master, then just copy the generated file into your nagios
conf.d directory, and check the config / restart your nagios.