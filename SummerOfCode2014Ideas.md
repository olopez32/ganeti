# Google Summer of Code 2014 #

Here are our project ideas for Google Summer of Code 2014

It might be interesting to have a look at [last year's ideas](SummerOfCode2013Ideas.md) as well.

# Project proposals #

## Improvements in the monitoring daemon ##
(language: haskell) [design](http://docs.ganeti.org/ganeti/master/html/design-monitoring-agent.html)

  * Implement additional data collectors, for example:
    * [Ganeti daemon status](http://docs.ganeti.org/ganeti/master/html/design-monitoring-agent.html#id13)
    * [Hypervisor resources report](http://docs.ganeti.org/ganeti/master/html/design-monitoring-agent.html#id14)
    * [Node OS resources report](http://docs.ganeti.org/ganeti/master/html/design-monitoring-agent.html#id15)
    * LVM
      * physical volumes collector
      * volume groups collector
    * Ceph or Gluster collectors
    * Network performance collector
    * ... and we can consider any other collector you might be interested in
  * Optimize the currently existing collectors
  * Implement cache system to decouple between collection and retrieval
  * Allow different stateful collectors to be queried with different time intervals
  * Implement plugin interface
  * Use dynamic memory usage information in hbal/hail

## Improvements to the instance bin-packing tool (hsqueeze) ##
(language: haskell) [design](http://docs.ganeti.org/ganeti/master/html/design-hsqueeze.html)

  * Add -X option to hsqueeze (actually executing the derived packing)
  * Make hsqueeze use the dynamic CPU data that is now collected thanks to last year's summer of code project

## Configuration daemon ##
(language: haskell)
[https://code.google.com/p/ganeti/issues/detail?id=564 Allow generic queries in ConfD)

Ganeti confd supports a restrictive set of queries: we'd like to expand its language to be able to query any configuration value from the ganeti config. Example interesting values could be names and UUIDs for nics and disks, network information, availability of IP addresses in pools.


## Improve hypervisor support ##
(language: python)

  * Support for kvm+cgroups for strict resource allocation
  * Support for kvm+namespaces for added security
  * Improve linux containers support
  * Support for hugepages for kvm

## Location awareness ##
(language: haskell)

  * Improve htools (hail, hbal, hsqueeze etc) to provide location awareness, as discussed in GanetiCon.
  * This would involve modeling better the relationships between different nodes and nodegroups (are they close or far latency-wise? is there a bottleneck?) and instances (which ones are redundancy-ones in a pool? which ones should live close to each other?) to provide better "service-level" allocation instead of simple per-instance one.
  * This feature would allow to have primary and secondary nodes on different nodegroups, but also to specify which instances should and should not share point of failures (nodegroup, node) as much as possible.

## Improve containers support ##
(language: python)

  * Work with docker.io and LXC to provide strong ganeti-based linux containers
  * Solve container problems like startup, storage, networking, which are different from how traditional VMs work

## KVM resource constrainment ##
(language: python)

Objectives:

  * cgroups integration for more reliable resource constrainment
  * linux containers integration for better control of what the qemu executable can access

Benefits:

  * better segregation in case of virtualization/qemu breakup
  * better resource accounting

## Conversion between disk templates ##
(language: python)

Currently, Ganeti supports only disk conversions between LVM and DRBD. Introduce a generic mechanism to convert between other templates too. This mechanism will need to copy the disks' data for conversion to be possible.

## Authnz support for RAPI ##
(language: python)

Currently the RAPI supports only coarse-grained permissions (read/write). It is easy to implement more fine-grained access control by implementing a pluggable authentication and authorization API. The authentication API will allow authenticating users against different backends, while the authorization API will authorize individual OpCodes before they are submitted to the job queue.