  * This is work in progress, to be peer reviewed
  * (Initial document based on 1 experience and based on some disparate notes. Don't just apply these settings without extensive testing!)

# Introduction #

This page should evolve to a best practice guide on how to tune different parts of a ganeti cluster, for better performance.

# KVM #

## Host CPU ##

Passes the Host CPU through to the VM, allowing the VM to use all the
features the host CPU has, speeding up certain workloads (like using aes-ni for faster crypto).

```
    gnt-cluster modify -H kvm:cpu_type=host      # entire cluster
    gnt-instance modify -H cpu_type=host [vm]    # single VM
```

In case that your cluster has nodes with different CPUs, note that a live migration between two different CPU types could crash the instance. Ganeti will not warn you about this at the moment, so you will have to handle the migrations between such nodes yourself. Although Ganeti could provide support for this in the future, it does not at the moment. See issue <a href='https://code.google.com/p/ganeti/issues/detail?id=895'>895</a>.

## KSM ##

KSM periodically merges same memory pages, increasing the amount of available memory. While Ganeti does not explicitly support KSM or account for memory oversubscription, KSM can be used to relieve memory pressure.

To activate:

```
echo 1 > /sys/kernel/mm/ksm/run
echo 100 > /sys/kernel/mm/ksm/sleep_millisecs
```

To check status:

```
grep "" /sys/kernel/mm/ksm/*
> /sys/kernel/mm/ksm/full_scans:203
> /sys/kernel/mm/ksm/pages_shared:769324
> /sys/kernel/mm/ksm/pages_sharing:1510361
> /sys/kernel/mm/ksm/pages_to_scan:100
> /sys/kernel/mm/ksm/pages_unshared:4712835
> /sys/kernel/mm/ksm/pages_volatile:602177
> /sys/kernel/mm/ksm/run:1
> /sys/kernel/mm/ksm/sleep_millisecs:20
```

# DRBD #
See also [Issue 57](https://code.google.com/p/ganeti/issues/detail?id=57)

## latency tuning ##
As per http://www.drbd.org/users-guide/s-latency-tuning.html :

  1. use a dedicated nic for DRBD replication and configure jumbo frames: set the mtu on that interface as high as possible 7200-9000 -
  1. use the deadline scheduler on the disks used as PV's for the Volume Group(s) used by DRBD

```
gnt-cluster command aptitude install sysfsutils
cat <<EOF >>/etc/sysfs.conf
block/sda/queue/scheduler = deadline
block/sda/queue/iosched/front_merges = 0
block/sda/queue/iosched/read_expire = 150
block/sda/queue/iosched/write_expire = 1500
EOF
gnt-cluster copyfile /etc/sysfs.conf
gnt-cluster command /etc/init.d/sysfsutils restart

cat <<EOF >> /etc/sysctl.d/60-drbd-tuning.conf
# Increase "minimum" (and default) 
# tcp buffer to increase the chance to make progress in IO via tcp, 
# even under memory pressure. 
# These numbers need to be confirmed - probably a bad example.
#net.ipv4.tcp_rmem = 131072 131072 10485760 
#net.ipv4.tcp_wmem = 131072 131072 10485760 
# reduce water levels to start marketing background (and foreground) 
# write back early. Reduces the chance of resource starvation. 
#vm.dirty_ratio = 10
#vm.dirty_background_ratio = 4
EOF
gnt-cluster copyfile  /etc/sysctl.d/60-drbd-tuning.conf
gnt-cluster command "cat /etc/sysctl.d/*.conf /etc/sysctl.conf | sysctl -p -"
```

# Networking #

Lari Hotari reported on the list that to fix a network performance problem he added the following lines to the intefaces file:

```
pre-up /sbin/ethtool --offload br0 gso off tso off sg off gro off
pre-up /sbin/ethtool --offload eth0 gso off tso off sg off gro off
```

(where br0 is the instance bridge, and eth0 the physical network interface).

Reference: [RHEL Virtualization Host Configuration](https://access.redhat.com/knowledge/docs/en-US/Red_Hat_Enterprise_Linux/6/html/Virtualization_Host_Configuration_and_Guest_Installation_Guide/ch10s04.html)

This should apply to both KVM and Xen.