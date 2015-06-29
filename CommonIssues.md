<h1> Introduction </h1>
This document provides troubleshooting tips for issues commonly encountered by Ganeti users.

<h2>Table of contents</h2>


# Networking problems #

## Instances aren't reachable ##

If the instances you create are unreachable, you may have one of two main problems:
  * Networking hasn’t been configured on the OS of the instance. Depending on which hypervisor you’re using, use `gnt-instance console` or connect to the VNC console to access to the instance and configure its networking.
  * The network configuration on the host or on Ganeti is not correct. Check all of the instance’s network interface settings to make sure they’re correct. If applicable, check to see if the host’s IP forwarding and iptables rules are correctly set.

## `gnt-cluster init` fails complaining with the error message "Cluster IP already active" ##

Each Ganeti cluster of _n_ nodes needs at least _n+1_ IP addresses: one IP address for each node, plus an extra IP address that represents the cluster itself and that is “floating” across the nodes. Specifically, the cluster address is active on a network interface of the master node, and it is migrated to the new master when the master node is failed over.

`gnt-cluster init` tries to take ownership of this IP address and assign it to the master of the cluster being initialized. Before taking ownership of the address, this command checks whether the address is already active (as it should _not_ be active). If the IP address is already active on a machine, the error message “Cluster IP already active” is triggered.

To resolve this error:
  1. Make sure that the IP is actually the IP that needs to become the cluster IP, as opposed to the IP of the main network interface of one node.
  1. If the IP address is in fact the correct IP, find the machine on which that IP (`$MASTER_IP`) is active, and the interface (`$MASTER_NETDEV`) to which the IP is assigned.
  1. On said machine, execute the following command as root:
```
ip addr del "$MASTER_IP" dev "$MASTER_NETDEV"
```
  1. Run `gnt-cluster` init on the master to initialize the cluster.

## DHCP: Best Practices ##

  * Currently, Ganeti does not ship a DHCP server. Therefore, a custom DHCP server must be employed.
  * To start/configure a DHCP server, there are several hooks which can be used to customize network interfaces through scripts.
  * Instance MAC addresses and IP addresses should be unique within a single host. Otherwise, NAT through iptables can be used to translate instance IP addresses to unique IP addresses within the host.
  * The [Ganeti OS installation redesign](http://docs.ganeti.org/ganeti/master/html/design-os.html) design doc employs a DHCP server to assign IP addresses to instances. This doc is a good starting point for learning more about using DHCP with Ganeti.

# Instance console issues #

**To exit the console at any time:** Type `CTRL+]`. This method works on both Xen and KVM.

**If nothing is displayed on the console:** Try pressing `Enter`.

**If the console doesn’t work:**
  * Double-check that the master can log in to the node as root, without a password.
  * Check that your instance configuration has a “getty” on the right port. For example:
    * On KVM, you need a getty on ttyS0
    * On Xen pvm, you need a getty on hvc0
    * How you configure this option depends on your distribution. See examples for Debian and Ubuntu on Ganeti's [instance-debootstrap os](http://git.ganeti.org/?p=instance-debootstrap.git;a=blob;f=create;h=c276b042daa190899cbfecff26fa8dfb4ad93034;hb=HEAD).
  * If you added the correct line to `inittab`, `/etc/init`, or `/etc/event.d`, remember to restart init (you can use `killall -HUP init`) or reboot the instance to verify that your setup works.
  * If you have absolutely no instance access, try stopping the instance using `gnt-instance activate disks`, and then mounting the disks to add the relevant configuration.

# Instance disk access #

Because all instances are different, some of these steps might be redundant for your instance. If the instance is running LVM or some other peculiar config, you may need to take additional steps. The following commands work for a standard instance with a partition inside a block device:

  1. Stop the instance.
  1. Run `gnt-instance activate-disks <instance-name>` and note the path of the device (the path after the last “:”).
  1. If the device is a file, run `losetup -f <file>` , and then use `losetup -a`  to find the correct device.
  1. Run `kpartx -a <device>`  (this is either the activate-disks device or the losetup device).
  1. Run `mount` on `/dev/mapper/<device>p<int>`, substituting the appropriate variables for the device name and the partition number you want to access.
  1. Perform the previous steps in reverse order:
    1. `umount`
    1. `kpartx -d <device>`
    1. `losetup -d /dev/loop<int>`  (If you used `losetup`  before)
    1. `gnt-instance deactivate-disks`  (Optional; use if you plan to start the instance immediately)


# Instance migration/move problems #

## The instance move is taking too long. ##
All traffic in inter- and intra-cluster moves is transferred using socat and encrypted by default. In intra-cluster moves, the encryption tends to be the limiting factor for the speed of moves. Allowing more options for encryption, including no encryption at all, is a planned feature for 2.12.

If security is of no concern, one useful trick is to convert the instance to use a disk template with redundancy (e.g. DRBD). Failing the instance over and changing disk templates again can significantly outperform a standard instance move.

For moves in which the speed of the network connection is the problem, try using the `--compress` option provided by all operations performing instance moves in 2.10. This option can help reduce the amount of data sent over the network by compressing the instance image.

## A cross-cluster instance move is preventing other actions. ##
Unfortunately, this is a known issue that will be addressed in Ganeti 2.12 by using opportunistic locking in instance moves.

## I receive a certificate error when trying to start an instance move. ##
The move-instance tool uses RAPI, which requires the _rapi.pem_ certificate file to be passed to it as an argument. If the instance is being moved between clusters, both of the clusters’ RAPI certificate files must be provided.

## I have some other mysterious issue with instance moves/migration. ##
We recommend first checking the import/export scripts of the used OS image. Ganeti uses these scripts to perform moves and migrations, and the scripts often go untested prior to events such as moves and migrations.

# Upgrades: Best practices #

## Ganeti versions: ##
  * If the version of Ganeti you’re upgrading is version 2.10 or higher, you can simply use `gnt-cluster upgrade --to=2.xx` on the master node.
  * For older versions, follow Ganeti’s  Upgrade notes.
**Note:** Instances can remain alive during an upgrade.

## Node OS ##
  * If your cluster has at least 3 nodes using DRBD, the safest way to upgrade is node by node, upgrading the master node last:
    1. Remove a node from the cluster
```
gnt-node modify -D yes $NODE
hbal -L -X
gnt-node modify -O yes $NODE
```
    1. Upgrade the node OS.
    1. Re-add the node to the cluster:
```
gnt-node add --readd $NODE
```
  * If you have sufficient resources, you can also set up a new cluster with the new OS system, and then use the inter-cluster instance move to transfer the instances.

## DRBD versions ##
While we don’t recommend using different DRBD versions within a single node group for an extended period of time, Ganeti still works reliably with an non-homogenous DRBD setup during the upgrade process. The safest way to upgrade DRBD versions is node by node, upgrading the master node last:
  1. Remove a node from the cluster:
```
gnt-node modify -D yes $NODE
hbal -L -X
gnt-node modify -O yes $NODE
```
  1. Upgrade the node os.
  1. Re-add the node to the cluster:
```
gnt-node add --readd $NODE
```

# Snapshots and backups #

## Does Ganeti support LVM snapshots?/How do I make a LVM snapshot in Ganeti? ##
There is some confusion regarding the use of LVM snapshots in Ganeti. Ganeti does not support LVM snapshots, in the sense that it doesn’t support creation of minimal-size snapshots that persist, grow as needed, and that can be used to restore a state. Lack of support for these types of LVM snapshopts is due to the slowdown users experience after significant changes are made and the LVM snapshots grow too large.

Ganeti _does_ use LVM snapshots to create a stable view of a volume currently in use which needs to be backed up, but the snapshot is deleted once a backup is made.

## How do I make a backup of an instance? ##
The `gnt-backup export` command can be used to export an instance to any node in the cluster. The backup contains the data and the configuration of the instance, and can be found in the `/srv/ganeti/export/$instance` directory.

## I’m experiencing a mysterious issue with gnt-backup. ##
Much like in the case of instance moves, export and import scripts are often to blame for strange behavior, especially if said scripts are untested beforehand. We recommend first checking the import/export scripts of the used OS image.

# RBD/Ceph: Best practices for installation and setup #

Ganeti doesn’t require any kind of special Ceph configuration. To deploy and configure RBD/Ceph:
  1. Follow the deployment and configuration instructions on the [Ceph website](http://ceph.com/docs/master/start/). Ganeti doesn’t use the Ceph Filesystem or MDSes, so you can skip these sections of Ceph’s instructions.
  1. Once Ceph is up and running and you configure a RADOS block device storage pool for Ganeti (by default named `rbd`), tell Ganeti to use RBD:
```
gnt-cluster modify --enabled-disk-templates rbd \
          	             --ipolicy-disk-templates rbd
```
  1. Now that RBD is enabled, specify the pool Ganeti should use. The default value is `rbd`, so on a fresh cluster, this step is a no-op.
```
gnt-cluster modify -D rbd:pool=rbd
```
  1. Configure Ceph on all Ganeti notes by following the [Installing RBD](http://docs.ganeti.org/ganeti/master/html/install.html#installing-rbd) instructions of the Ganeti installation tutorial. For example:
```
# This wil run the same command on all nodes in your cluster.
# NOTE: On Ganeti nodes, /etc/ceph/ceph.conf must at least enumerate the
#       IP addresses and ports of all Ceph monitors.
dsh -Mf /var/lib/ganeti/ssconf_node_list \
  "apt-get update;
   apt-get install ceph-common;
   scp $HOSTNAME:/etc/ceph/ceph.conf /etc/ceph/"
```
  1. Verify that all nodes can access Ceph. If this command completes on all nodes, you're good to go.
```
dsh -Mf /var/lib/ganeti/ssconf_node_list rbd list
```
  1. If all has gone well up to this point, you can start your first RBD instance. For example:
```
gnt-instance add -t rbd -s 80G helloworld.example.org
```
  1. If you’re using Ganeti 2.10 or newer and KVM, you can exploit its native support for Ceph and get a free performance boost by enabling userspace support:
```
gnt-cluster modify -D rbd:access=userspace
```

**Note:** Gluster configuration is very similar. To configure Gluster:
  1. Deploy Gluster.
  1. Install client-side support for Gluster on all nodes.
  1. Enable the Gluster disk template.
  1. Configure the remote host and volume Ganeti should use.

# Instance installation problems #

## OS installation problems ##
See the [Ganeti OS installation redesign](http://docs.ganeti.org/ganeti/master/html/design-os.html) document for troubleshooting information.

## How do I set a root password/key? How do I set a root password/key with hooks? ##

Setting a root password or key for the OS you're installing is highly dependent on the OS scripts you use. Here’s an example related to the instance-debootstrap OS install scripts:

By default, `instance-debootstrap` resets the root password so that newly-created instances have an empty password. Therefore, you don't need any special steps to provide access to the instance.

If you need a password immediately for the instance as it’s being created, you have to use a hook. The complete instructions are in the [README file of the instance-debootstrap package](http://git.ganeti.org/?p=instance-debootstrap.git;a=blob_plain;f=README;hb=HEAD). The process can be summarized as follows:
  1. Copy the file `examples/hooks/defaultpasswords` to `$sysconfdir/ganeti/instance-debootstrap/hooks/`.
  1. Copy the data file `examples/hooks/confdata/defaultpasswords` to `$sysconfdir/ganeti/instance-debootstrap/hooks/confdata/`.
  1. Modify the data file accordingly to your needs. The file syntax is such that each line represents a user, with the format `username:password`.
  1. After copying the two files, run the instance creation as usual to automatically execute the files.

## Selecting a kernel for the instances ##

The kernel is specified in the hypervisor parameters and can be modified. For example:
```
gnt-cluster modify -H 
kvm:kernel_path=/boot/vmlinuz-2.6-kvmU,initrd_path=/boot/initrd-2.6-kvmU
```


# Failure scenarios and recovering from failure scenarios #

## Master failures/After a failure, two nodes think they are master ##
When the master node fails to connect to the rest of the network because of a network failure, you simply need to fix the network. Once the network is fixed, the master recovers automatically.

In the case of a more serious failure, a master failover is needed. A master failover must be triggered manually. To perform a master failover:

  1. Make sure that the original failed master won't start again while a new master is present, preferably by physically shutting down the node.
  1. To upgrade one of the master candidates to the master, issue the following command on the machine you intend to be the new master:
```
gnt-cluster master-failover
```
  1. Offline the old master so the new master doesn't try to communicate with it. Issue the following command:
```
gnt-node modify --offline yes oldmaster
```
  1. If there were any DRBD instances on the old master node, they can be failed over by issuing the following commands:
```
gnt-node evacuate -s oldmaster
gnt-node evacuate -p oldmaster
```
  1. Any _plain_ instances on the old master need to be recreated again.

## Re-adding nodes ##
When a failed node (either a regular or master node) is repaired and ready to be added to the cluster, reinstall Ganeti on the node and then re-add it to the cluster using the following command:
```
gnt-node add --readd nodename
```

After re-adding a node, it's a good idea to run `hbal --luxi --print-commands` on the master node to obtain the list of commands to balance the cluster and populate the new node with instances. Running `hbal --luxi --exec` executes the commands directly.

## Particular case: 2 node cluster ##
If the master node fails on a 2 node cluster, upgrading the non-master node to become the new master requires special handling. This is because a master needs to obtain a majority of votes from the rest of the network to ensure there is no other master running. In the case of a cluster with only two nodes, neither node can obtain the majority.

To fail over the master node on a 2 node cluster:
  1. Issue the following command on the new master:
```
gnt-cluster master-failover --no-voting
```
  1. Manually start the master daemon with the following command:
```
ganeti-masterd --no-voting
```
  1. Run the following command to ensure that the cluster is consistent again:
```
gnt-cluster redist-conf
```

## Ganeti’s autorepair (in)capabilities ##
Ganeti’s _ganeti-watcher_ daemon makes sure that all instances marked as up are running. It also reactivates secondary DRBD block devices of instances on nodes that are rebooted. In sum, it tries to ensure that if a node is rebooted, all affected instances are eventually fixed and brought to their original state.

Ganeti doesn’t perform failovers automatically; they must be triggered manually. However, there are tools that help the administrator perform common related tasks:
  * `gnt-node evacuate`: Moves instances from a given node
  * `hbal`: Automates the task of distributing instances evenly on the nodes. Displays a list of recommended commands or performs them automatically.

# DRBD/storage problems #

## Degraded/unsynced disks ##
DRBD disks can experience various error states, some of which Ganeti can recover from (semi-) automatically.
  * If the connection between the primary and secondary DRBD node is lost, Ganeti automatically reconnects the DRBD pair with the (regularly executed) Ganeti watcher. If you want to manually reconnect the DRBD pair, you can issue the command `gnt-instance activate-disks` for the instance (even while the instance is running). DRBD automatically syncs the changed portions of the disks from the primary node to the secondary node.
  * If disks (either primary or secondary) fail and have to be replaced, you can use `gnt-instance replace-disks` to recreate and resync those disks and return them to a fully functional and replicated state.

## Performance problems ##
There is no single source of DRBD performance problems, and thus, no single solution. There are a few good starting points for diagnosing such problems:
  * Make sure that you have a dedicated replication network configured by specifying the proper secondary IPs for all nodes in the cluster.
  * Verify the DRBD-related disk parameter documented in the [gnt-cluster man page](http://docs.ganeti.org/ganeti/current/html/man-gnt-cluster.html) (especially `resync-rate`, `protocol`, and the `dynamic-resync`-related parameters).
  * Refer to the [Optimizing DRBD performance](http://www.drbd.org/users-guide/p-performance.html) guide.

## Maximum size of disks ##
The maximum size of DRBD devices which can be created through Ganeti is 4TB (see [Issue 256: DRBD Volume above 4TB not working](https://code.google.com/p/ganeti/issues/detail?id=256)).

To create larger DRBD devices, you must set up the devices manually and use them via the blockdev disk template. However, you will lose all automatic DRBD management performed by Ganeti, as well as the possibility to failover/migrate instances to their secondary node.

## Converting disk templates ##
Ganeti can convert between DRBD and plain LVM volumes. To accomplish this conversion, use `gnt-instance modify` and provide the desired new disk template with the `--disk-template` command line parameter. Refer to the [gnt-instance man page](http://docs.ganeti.org/ganeti/current/html/man-gnt-instance.html) for further details. Ganeti does not support any other disk template conversion.

## drbd8-utils and Ubuntu ##
Upgrades to the drbd8-utils package in Ubuntu have resulted in some DRBD 8.4 syntax requirements spilling into DRBD 8.3 tools as well. Ganeti's attempts to execute previously valid commands fail unless the compatibility executable is used instead of the one linked by default. A temporary way to resolve this:

```
mv /sbin/drbdsetup /sbin/drbdsetup84
ln -s /lib/drbd/drbdsetup-83 /sbin/drbdsetup
```

For more details, look at the forum topic:
https://groups.google.com/forum/#!msg/ganeti/MkCNmzF6hu8/kTPOELyEkdsJ

# Cleanup issues #

## gnt-node add fails when an old cluster wasn't cleaned up properly ##

The message `Unhandled Ganeti error: Given cluster certificate does not match local key` can indicate that an old `/var/lib/ganeti/server.pem` certificate (on the node to be added) still exists. If you ran a cluster cleanup script to wipe your cluster, make sure to run it on all nodes that shall be added to the new cluster.

# htools: Debugging hail/hbal #

Tips and tricks for debugging hail and hbal:
  * Before debugging hail, verify that the dom0 resources (in particular, memory) are set to reasonable values. If hail can’t allocate new instances, it’s very likely that dom0 is using an unreasonably large amount of a node’s memory.
  * hail speaks the iallocator protocol (for more information, see [Ganeti automatic instance allocation](http://docs.ganeti.org/ganeti/current/html/iallocator.html)), so debugging hail is possible independent of any cluster.
  * hail supports the text format via the `-t` option, which provides the cluster configuration in text format. You can generate a text format description of the current live state of the cluster by running `hbal -L -S config`, which creates a file named _config.original_.
  * hbal also supports the `-t` option and therefore can be debugged in a way similar to hail.
  * To verify what data is passed to hail by `gnt-instance add`, use the `-I` option to specify your own instance allocator. This option can also be a script that saves the input and returns a constant output.
  * To determine the largest instance size that still fits onto the cluster according to htool’s understanding, use tiered allocation, as provided by `hspace -L`.

# Using gnt-`*` commands #

Tips for gnt-`*` commands:

  * Commas inside parameters have to be escaped with a backslash, because Ganeti uses commas as separators.

# Other helpful documentation #

Here you find some documentation that Ganeti users provided:

  * [A Ganeti cheat sheet](https://nsrc.org/workshops/2014/sanog23-virtualization/raw-attachment/wiki/Agenda/ganeti.pdf)
  * https://nsrc.org/workshops/2014/sanog23-virtualization/raw-attachment/wiki/Agenda/ganeti-settings.pdf