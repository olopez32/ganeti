These are the features that we aim to add before the final 1.2 release:
  * remote\_raid1 template consistency recovery after secondary node reboot
  * allow node failover (i.e. all primary instances on a node are failed over to their secondary node); this is just a batch form for `gnt-instance failover`
  * allow node evacuation (i.e. all secondary instances on a node will have their secondary replaced); this is just a batch form for `gnt-instance replace-disks`
  * expose tag support to the command line tools (`gnt-{cluster,node,instance}`)
  * instance reboot: more powerful reboot functionality: at instance OS level, at hypervisor level or at ganeti level (this one is equivalent to `gnt-instance shutdown` + `gnt-instance startup`)