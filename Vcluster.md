# Vcluster #

vcluster, meaning "virtual cluster", is a setup where one physical machine pretends to be several Ganeti nodes (by spawning lots of node daemons). It is useful to test stuff which require large clusters to be tested properly.

See the [design doc](http://git.ganeti.org/?p=ganeti.git;a=blob_plain;f=doc/design-virtual-clusters.rst;hb=HEAD) for details.

This wiki page just summarizes the steps that are necessary to set up a vcluster (for example to run a QA) on your own machine.

## Cleaning up previous vclusters ##

Clean up rests of previous vclusters:

Delete the vcluster installation files (depends on how you configured the previous installation):

```
rm -rf /srv/ganeti/vcluster/* 
```

Remove network interfaces that might still be there (depends on how you configured them before):

```
ifconfig lo:vcluster:0 down
ifconfig lo:0 down
```

## Setting up the machine ##

Setup the vcluster infrastructure by invoking the `vcluster-setup` tool. It is located in the `ganeti/tools/` directory of your installation. You need to specify the path of the vcluster installation (for example `/srv/ganeti/vcluster`.

```
mkdir -p /srv/ganeti/vcluster/
[...]/lib/ganeti/tools/vcluster-setup -c 15 -i 9 -n lo:vcluster
```

The option `-c` enumerates the number of 'virtual' nodes and `-i` the number of instances you intend to use on the vcluster. Make sure that this complies the the configuration of your vcluster QA file. `-n` specifies the name of the network interface. It will be created by the setup, you do not need to set it up yourself.

## The QA config ##

For a QA to run, you need to configure it with certain limitations. See here some annotated excerpts from a config file:

Name the cluster 'cluster', omit/disregard the rename-name, because renaming does not work on vclusters anyway.

```
{
  "name": "cluster",
  "#rename": "disabled",
```

Specify the host name of the physical machine that runs the vcluster and the path where you set up the vcluster.

```
  "# Virtual cluster": null,
  "vcluster-master": "hostname.of.your.machine",
  "vcluster-basedir": "/srv/ganeti/vcluster",
```

Specify the following init args so that the QA does not try to edit the /etc/hosts and that drbd-storage is disabled. In newer versions, the option `--no-drbd-storage` needs to be replaced by `--enabled-disk-templates=diskless`.

```
  "cluster-init-args": ["--no-etc-hosts", "--no-drbd-storage"],

  "enabled-hypervisors": "fake",
  "enabled-disk-templates": "diskless",
```

Various parameters that have shown to be reasonable.

```
  "hypervisor-parameters": {},
  "primary_ip_version": 4,

  "os": "busybox",
  "mem": "600M",
  "maxmem": "600M",
  "minmem": "500M",

  "# Lists of disk sizes": null,
  "disk": ["512M"],
  "disk-growth": ["512M"],

  "# Instance policy specs": null,
  "ispec_mem_size_max": 600,
  "ispec_disk_size_min": 0,
  "ispec_disk_count_min": 0,
```

Specify the nic that you configured the vcluster setup with.

```
  "master-netdev": "lo:vcluster",
  "default-nicparams": { "mode": "bridged", "link": "yourbridgename" },
```

Specify up to as many but not more nodes than you specified in the vcluster configuration on the machine. Use actually the names ```nodeX``` here, because the cluster setup assumes those names. Pick the IPs that the vcluster configuration put into the `/etc/hosts` file on the machine.

```
  "# Up to 90 nodes would be possible, but gnt-cluster verify can't handle more than this": null,
  "nodes": [
    {
      "# Master node": null,
      "primary": "node1",
      "secondary": "192.0.2.10"
    },

    {
      "primary": "node2",
      "secondary": "192.0.2.11"
    },

 ...

    {
      "primary": "node15",
      "secondary": "192.0.2.24"
    }
  ],

```

Specify up to as many instances (but not more) than you configured the vcluster with. Use actually the instance names `instanceX` here.

```
  "instances": [
    { "name": "instance1" },
    { "name": "instance2" },
...
    { "name": "instance9" }
  ],
```

For the tests, there are a couple of tests that don't work on vclusters. This is subject to change, so don't assume that this list is exhaustive or correct.

```

  "tests": {
    "default": true,
    "default-instance-tests": true,

    "# NOTE: The tests below are disabled because they do not work": null,
    "# properly in virtual clusters. Some could be made to work with": null,
    "# a bit of work on the QA code.": null,

    "cluster-burnin": false,
    "cluster-exclusive-storage": false,
    "cluster-reserved-lvs": false,
    "env": false,
    "group-custom-ssh-port": false,
    "haskell-confd": false,
    "htools": false,
    "instance-add-drbd-disk": false,
    "instance-add-plain-disk": false,
    "instance-export": false,
    "instance-import": false,
    "instance-modify": false,
    "instance-recreate-disks": false,
    "instance-remove-drbd-offline": false,
    "instance-rename": false,
    "exclusive-storage-instance-tests": false,
    "test-jobqueue": false,

    "instance-device-hotplug": false,

    "job-list": true,

  },
}
```

## Tips and Tricks ##

  * Fetching the logs: If you want to fetch the logs of a vcluster, don't forget that you need to fetch it from `/srv/ganeti/vcluster` (or whatever you configured) and not from `/var/log/ganeti`.