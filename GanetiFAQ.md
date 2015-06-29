

# Versions supported #

## Which versions of Ganeti are supported/maintained? ##

http://docs.ganeti.org/ lists the latest stable version of Ganeti. The core Ganeti team supports and maintains the two most recent stable versions of Ganeti.

However, the Ganeti team also fixes high severity bugs in older versions. The decision whether or not to fix a bug in an older version of Ganeti is made on a case-by-case basis, and depends on the complexity of the fix, the impact of the fix on existing users, and the risk of the fix introducing regressions.

In practical terms, our policy means that we support at minimum security issues in older versions of Ganeti shipped by some major Linux distributions (which usually means the last security-supported version is the one present in Debian stable). This support is bug-driven only.

## Which distributions have Ganeti available as packages? Are there RPMs for Ganeti? ##

The following distributions package Ganeti:
<table cellpadding='5' border='1'>
<tr><td><b>Distribution</b></td><td><b>Package name</b></td></tr>
<tr><td>Debian</td><td>
<ul>
<li><a href='http://packages.debian.org/wheezy/ganeti2'>Wheezy</a>: <font face='courier'>ganeti2</font></li>
<li><a href='http://packages.debian.org/jessie/ganeti2'>Jessie</a>: <font face='courier'>ganeti</font></li>
</ul>
We recommend using the newer Ganeti versions in <a href='http://packages.debian.org/wheezy-backports/ganeti2'>wheezy-backports</a>.</td>
</tr>

<tr><td>Ubuntu</td><td>
<ul>
<li>Up to <a href='http://packages.ubuntu.com/trusty/ganeti'>Saucy</a>: <font face='courier'>ganeti2</font></li>
<li>As of <a href='http://packages.ubuntu.com/trusty/ganeti'>Trusty</a>: <font face='courier'>ganeti</font></li>
</ul>
</td>
</tr>

<tr><<td>RPMs (RHEL/CentOS/Scientific Linux and Fedora)</td>
<td>See the current packaging effort at <a href='https://github.com/jfut/ganeti-rpm'>GitHub.</a></td>
</tr>

<tr><<td>Gentoo</td>
<td>See the current <a href='http://packages.gentoo.org/package/app-emulation/ganeti'>ebuild file.</a></td>
</tr>
</table>


## What guest operating systems does Ganeti support? ##

Ganeti does not place any restrictions on guest operating systems. However, the limitations of the selected hypervisor still apply (e.g. xen-pvm can’t run Windows guests).

Ganeti relies on user-provided OS installation scripts to install guest operating systems. These scripts allow for very flexible installation scenarios and are documented at http://docs.ganeti.org/. It is also possible to boot an instance from an image and then manually install the OS.

## Which versions of Xen does Ganeti support? ##

From Ganeti 2.6 onwards, the supported Xen versions are:
  * 3.0.3 and later 3.x versions
  * 4.x, tested up to 4.1
See the [Ganeti installation tutorial](http://docs.ganeti.org/ganeti/current/html/install.html#id2) for updates.

## Which Xen toolstacks are supported by Ganeti? ##

Ganeti supports the following Xen toolstacks:

| **Version** | **Xen toolstacks supported** |
|:------------|:-----------------------------|
| Before Ganeti 2.6 | **xm** only                  |
| Ganeti 2.6 and upwards | Both **xm** and **xl** toolstacks are supported, but must be specified at configure time |
| Ganeti 2.10 and upwards | **xl** toolstack is supported, with the option to switch between the toolstacks at runtime using a hypervisor parameter |

## What projects are related to/built on Ganeti?: ##

Below is a list of Ganeti-related projects of which we’re aware. Contact us if you want your project to be added!

<table cellpadding='5' border='1'>
<tr><td><b>Project type</b></td><td><b>Project</b></td></tr>
<tr><td>Cloud toolstack based on Ganeti</td>
<td>
<ul>
<li><a href='http://www.synnefo.org/'>Synnefo</a></li>
<li><a href='https://okeanos.grnet.gr/opensource/'>Okeanos</a> (public cloud based upon Synnefo)</li>
</ul>
</td>
</tr>

<tr><td>Web UI projects</td>
<td>
<ul>
<li><a href='http://ganeti-webmgr.readthedocs.org/en/latest/'>Ganeti Web Manager</a></li>
<li><a href='http://ganetimgr.readthedocs.org/en/latest/'>genetimgr</a></li>
</ul>
</td>
</tr>

<tr><td>Ganeti OS implementations</td>
<td>
<ul>
<li>instance-debootstrap: git://git.ganeti.org/instance-debootstrap.git (maintained by the Ganeti development team at Google)</li>
<li><a href='https://code.osuosl.org/projects/ganeti-image'>Ganeti instance image</a></li>
<li><a href='https://code.grnet.gr/projects/snf-image'>snf-image</a></li>
</ul>
</td>
</tr>
</table>
## What is the status of the SAN support of Ganeti? ##

SAN is fully supported in version 2.9., using the ExtStorage Interface. This interface  is a contribution from [GRNET](http://en.wikipedia.org/wiki/Greek_Research_and_Technology_Network). For more information, see:
  * The [Ganeti ExtStorage Interface slides](https://docs.google.com/a/google.com/file/d/0B934VF_cTqnwc0NGcEx4OXl3NDA/edit) from GanetiCon2013
  * The [Google Groups email thread](https://groups.google.com/forum/?fromgroups=#!searchin/ganeti-devel/shared-filer/ganeti-devel/kGTje2FP94k/in8s6RVFQqQJ) explaining how to set up the ExtStorage interface
  * The ExtStorage man page describes the interface and its usage further: http://docs.ganeti.org/ganeti/master/html/man-ganeti-extstorage-interface.html

You may need to write your own ExtStorageProvider scripts to comply with the interface.

# Google and Ganeti #

## What is the meaning of the word "Ganeti"? ##

The name "Ganeti" does not actually have a meaning. As one of the first team members of Ganeti put it "It was the best random name that we could come up with." It was chosen to be kind of unique in the sense of if you google for it, it will be the first hit, which it still is, this has nothing to do with that Google supports the project.

## Do the Google developers or a third party offer paid support for Ganeti? ##

Google does not offer paid support for Ganeti, nor do we know/recommend/endorse any third party that may do so. That being said, Ganeti the software is fully open and easily understandable by a good Linux admin/developer, and you can address questions to the following mailing lists:
  * User discussions: [ganeti@googlegroups.com](mailto:ganeti@googlegroups.com)
  * Ganeti development: [ganeti-devel@googlegroups.com](mailto:ganeti-devel@googlegroups.com)

## How is Ganeti used internally at Google? ##

Google uses Ganeti to provision virtual machines inside its corporate network. We have Ganeti clusters in offices and data centers, and use these clusters to provide basic network services, virtual workstations, and general Linux servers. Google doesn’t use Ganeti for production infrastructure.

## What are the future plans for Ganeti? ##

We’d like to scale Ganeti further (more nodes, instances, and parallel jobs) and to integrate interesting networking/virtualization/storage technologies as they become available. We don’t have a precise long term roadmap as of now.

# Best practices #

## How does Ganeti interact with other cluster solutions (OVF support, instance exports, etc.)? ##

You can easily convert a previously-exported Ganeti instance into an OVF package by using ovfconverter from the tools directory of the source code. The OVF package is supported by VMWare, VirtualBox, and some additional virtualization software.

You can also use an instance exported from a tool such as VMWare or VirtualBox and convert it to a Ganeti config file using the command `gnt-backup import`.

The document [Ganeti Instance Import/Export using Open Virtualization Format](http://docs.ganeti.org/ganeti/current/html/design-ovf-support.html) has a more detailed description of the internal design of the converter, as well as a list of the available command line options.

## How do I deploy Ganeti technically? ##
You can deploy Ganeti by following the [Ganeti installation tutorial](http://docs.ganeti.org/ganeti/master/html/install.html). The main steps include:
  * Installing the base operating system
  * Installing the required dependences
  * Installing Ganeti from the packages provided by the chosen Linux distribution (or compiling Ganeti from source code)
  * Installing at least one operating system support package
  * Initializing the cluster with gnt-cluster init and configuring the cluster using the appropriate `gnt-*` commands.
**Note:** The [Ganeti walk-through](http://docs.ganeti.org/ganeti/current/html/walkthrough.html) provides more information on initializing and configuring the cluster.

## What type of cluster users/deployment configurations does Ganeti serve well? ##

Ganeti is very suitable for small/medium deployments, and for starting small and then scaling up. It’s easy to start with even just 1 or 3 physical nodes, and then to add more nodes as needed. Managing a few clusters comprising up to 120-150 nodes should be easy for a good Linux admin or two capable of using configuration management (CFEngine, Puppet, Chef, etc.).

If you plan to scale your infrastructure to a large deployment (tens of clusters and thousands of nodes), management by someone with Python/Haskell knowledge, and the ability to use/deploy extra tools to coordinate such a big infrastructure, is advisable.


## What are known large and notable Ganeti deployments? ##

Large and notable Ganeti deployments include:

<table cellpadding='5' border='1'>
<tr><td>
<b>Deployment</b>
</td>
<td>
<b>Description</b>
</td></tr>
<tr>
<td><a href='http://www.debian.org/'>debian.org</a></td>
<td>Linux distribution<br>
<br>
5 servers, 70 VMs</td>
</tr>
<tr>
<td><a href='fsffrance.org/'>fsffrance.org</a></td>
<td>
See <a href='http://fsffrance.org/'>http://fsffrance.org/</a> for more details</td>
</tr>
<tr>
<td>Google</td>
<td>
Part of Google’s corporate computing infrastructure</td>
</tr>
<tr>
<td><a href='https://www.grnet.gr/'>grnet.gr</a></td>
<td>Greek Research & Technology Network<br>
<br>
10 clusters, >6000 VMs</td>
</tr>
<tr>
<td><a href='osuosl.org/'>osuosl.org</a></td>
<td>
Oregon State University Open Source Lab<br>
<br>
10 clusters, ~120 VMs</td>
</tr>
<tr>
<td><a href='http://www.skroutz.gr/'>skroutz.gr</a></td>
<td>
Price comparison engine<br>
<br>
20 nodes, >100 VMs, 3 locations</td>
</tr>
</table>

## Is automation of the Ganeti deployment process available? ##
There is no automated deployment process for Ganeti. However, since Ganeti version 2.11, you can upgrade an existing Ganeti cluster ( >2.10 ) using the single command: `gnt-cluster upgrade`.

## What are best practices for Ganeti network configuration? ##
The ideal Ganeti configuration requires two separate networks (or at least two separate VLANs):
  * **Primary network:** Includes all the nodes and provides connection to the external world
  * **Secondary network:** Used to replicate instance traffic across nodes

## How do you secure Ganeti instances, network-wise? ##
See [Security in Ganeti](http://docs.ganeti.org/ganeti/master/html/security.html) for security best practices and discussion of Ganeti components that require particular attention on matters security-related.

## How do you shield instances from each other? ##
Security-wise, disks of instances are separate logical volumes and are wiped before they’re attached to an instance. Other than separating instance disks, Ganeti relies on the hypervisor to separate instances. Newer versions of Ganeti (>= 2.12) also contain some measures to mitigate compromise of a node, should an instance manage to escape the hypervisor. See [Security in Ganeti](http://docs.ganeti.org/ganeti/master/html/security.html) for more details.

Performance-wise, [Partitioned Ganeti](http://docs.ganeti.org/ganeti/current/html/design-partitioned.html) allows resources, up to the level of individual spindles, to be individually assigned to instances.

## Is it possible to have nested virtualization on Ganeti (i.e. to use Ganeti nodes as VMs themselves)? ##
Yes. Ganeti itself makes no assumption about running on real hardware. Of course, in order to use nested virtualization, you must use virtualization solutions that are able to run inside each other. KVM at all levels is one popular nested virtualization solution. In fact, some developers use such a KVM setup for developing Ganeti. Additionally, the Ganeti buildbot ([buildbot.ganeti.org](http://buildbot.ganeti.org/)) runs completely on Ganeti VMs and includes a proper (KVM-based) QA.

## What are best practices for setting up a cluster with respect to (geo) redundancy? ##
Ganeti has no concept of geo location; the most closely-related concept to geo location is a node group. Nodes in any particular node group are assumed to be equivalent for usage, which means that they’re equivalent to existing in the same location. Therefore, if two nodes require geo redundancy, they need to be located in separate node groups. Instances are only moved between node groups upon explicit request, and deciding how to allocate instances among geo locations is the user’s responsibility.

To avoid two instances that provide the same service (such as DNS) from ending up on the same physical node, we recommend that you use exclusion tags. For more information, see the [hbal man page](http://docs.ganeti.org/ganeti/current/html/man-hbal.html).

## What are best practices for setting up monitoring/Nagios with Ganeti? ##
The health status of a Ganeti cluster can be verified with the command `gnt-cluster verify`. Ganeti-specific information can be obtained via the [ganeti-mond](http://docs.ganeti.org/ganeti/current/html/man-ganeti-mond.html). Individual Ganeti instances are best monitored independently of Ganeti with the same setup you’d use to monitor a physical machine.

## What are best practices for setting up configuration management (Puppet) with Ganeti? ##
Answer: It’s a good idea to use configuration management to fully and automatically install nodes, as well as to maintain consistent node configuration across a cluster. In particular, we recommend configuring specific (rather than minimal) versions to avoid surprises with available updates.

# Features: Storage #

## What distributed storage models does Ganeti support? ##

Ganeti introduced native [Ceph RBD](http://ceph.com/ceph-storage/block-storage/) support in Ganeti version 2.6. Native support for [Gluster](http://www.gluster.org/about/) volumes will be introduced in Ganeti 2.11. If the performance tradeoffs of true distributed storage are unacceptable to you, Ganeti also has excellent support for the limited-replica [DRBD](http://www.drbd.org/) technology.

Users of Ganeti 2.7 and newer versions may use the [external storage backend](http://docs.ganeti.org/ganeti/master/html/man-ganeti-extstorage-interface.html) to provide support for other storage systems, distributed or otherwise. You can also support distributed storage models through the "shared file" mechanism, so long as you can provide transparent replication of the shared file storage directory across all nodes (for example, with an NFS mount point).

In all cases, storage deployment and configuration is a task left to the administrator. Ganeti is simply configured to use an existing RBD pool, Gluster volume, or file system directory for instance data storage.

For more details on Ganeti and storage, see the [Ganeti administrator’s guide](http://docs.ganeti.org/ganeti/master/html/admin.html#disk-template) and the [Ganetti installation tutorial](http://docs.ganeti.org/ganeti/master/html/install.html#installing-rbd).

## Does Ganeti support backups and snapshots? ##

Ganeti supports backup of instances. Support for backup compression will be added in the upcoming Ganeti 2.10 release. Instance snapshotting is not yet available.

## What is the current status of distributed storage support? ##

As of January 2014, support for Ceph is now mature. Gluster support was only recently introduced, and therefore may be rough in places.

## Does Ganeti support instances with disks of different types (for example, a DRBD and a shared file disk)? ##

Support for instances with disks of different types is a planned feature. This feature will be available subsequent to the upcoming 2.11 release.

## Does Ganeti support multiple volume groups / storage classes ##

Ganeti supports instances with multiple volume groups.  Simply create instances overriding the "vg" attribute for the disk parameter.  Examples on how to do this can be found in [Hochschule Darmstadt Wiki](https://wiki.fbihome.de/Netzwerk:VM).



# Features: Cloud #

## How does Ganeti compare to other cloud solutions (OpenStack, Cloudstack, OpenNebula, XenCluster, libvirt, etc.)? ##

In comparison to other cloud solutions, Ganeti is much more lightweight. Ganeti is quite simple to set up and has very few dependencies on the required infrastructure. For example, Ganeti doesn’t even require a database. Also, Ganeti’s scope is more limited than other cloud solutions: it is made to manage small- to medium-sized clusters, rather than huge clouds and doesn’t provide a management UI out of the box. For a more in-depth comparison of VMWare, OpenStack, and Ganeti, see the [IaaS presentation](http://www.slideshare.net/gpaterno1/comparing-iaas-vmware-vs-openstack-vs-googles-ganeti-28016375Comparing).

libvirt, on the other hand, is only a library which abstracts over different virtualization technologies. It does not provide any cluster management features. While Ganeti can be used as abstraction for the supported hypervisors, this functionality is not the main purpose of Ganeti.


# Features: Fault tolerance/high availability #

## How does Ganeti provide fault tolerance? ##
Ganeti supports redundant storage solutions, the most important of which is DRBD. Ganeti also distributes instances to ensure that the secondary nodes have enough memory to immediately start an instance, should the primary node break.

## Does Ganeti support automatic failover? ##
Yes, Ganeti supports automatic failover. Automatic failover is opt-in, as some Ganeti system administrators prefer to either manually respond to breakages or to delegate such responses to some other system to take actions via RAPI.

`harep` is Ganeti’s auto-repair tool. `harep` can fix DRBD breakage, migrate and failover instances, and (as a last resort) reinstall instances. `harep` only acts on clusters, node groups, or instances for which a tag is set allowing the tool to take action.

# Features: Misc #

## What interfaces can be used to manipulate Ganeti? ##

All of Ganeti’s functionality is exposed through a set of command line tools, which are comprehensively documented in their corresponding [man pages](http://docs.ganeti.org/ganeti/current/html/manpages.html).

A subset of Ganeti’s functionality is available through a web interface: the [Remote API (RAPI)](http://docs.ganeti.org/ganeti/current/html/rapi.html). RAPI provides a REST-ful view of all the entities in a cluster, allowing them to be manipulated and queried. To address security concerns, RAPI provides authentication and encryption.

## What Ganeti management consoles are available? ##

While Ganeti itself does not provide a web management console, outside developers who manage large Ganeti deployments have developed solutions for this purpose and have made them available to the public.

One simple solution is Oregon State University Open Source Lab’s [Ganeti Web Manager](https://github.com/osuosl/ganeti_webmgr), a Django-based web application that exposes the commands accessible via the RAPI in a user-friendly way.

[Synnefo](http://www.synnefo.org/) provides a more feature-rich UI; however, the UI relies on Synnefo abstractions and cannot be used to manage a plain Ganeti cluster.

## Does Ganeti support containers as a form of limited virtualization? ##

Yes. Ganeti supports [LXC](http://linuxcontainers.org/) and chroot-based environments, allowing instances to be created as though a real hypervisor were used.

## What are the size limitations in Ganeti? (Number of nodes, instances, size of disks, number of disks) ##

**Nodes:** This number can range from 1 to 150. If you’re implementing a large number of nodes, they should ideally span multiple node groups.

**Instances:** This number is difficult to recommend, as it depends upon the specification of the node, the size of the instance, etc.

**Size and number of disks:** These numbers are difficult to recommend, as they depend upon the resources available in the nodes.

## What is the status of Open vSwitch? ##

There is a design document describing the integration of Ganeti and Open vSwitch (see [doc/html/design-openvswitch.html](http://docs.ganeti.org/ganeti/master/html/design-openvswitch.html)). This design document is still being implemented; currently, Ganeti can configure an Open vSwitch switch on nodes and also associate VLANs to instances’ NICs.

There have also been external experiments integrating Ganeti and Open vSwitch, most notably, the [AFOYI experiment](https://afoyi.com/blog/converting-ganeti-to-use-open-vswitch/) and [integration of KVM and Open vSwitch](https://afoyi.com/blog/converting-ganeti-to-use-open-vswitch/).

## What is the status of USB support? What are USB best practices? ##

USB devices can be passed directly to KVM instances when they are created (see man/gnt-instance.html for more information). For Xen, Ganeti currently simply enables USB support for domU, but does not add any USB devices. It is possible to add USB devices manually from dom0 (see [Xen USB Passthrough](http://wiki.xen.org/wiki/Xen_USB_Passthrough) for details). Patches for USB device support for Xen are welcome.

## What's the status of instance OS image support? ##

OS image support is currently under development and planned for release as part of Ganeti 2.11. See doc/html/design-os.html for a description of how OS images can be used as part of the OS installation for instances.




# Design decisions #

## Why does Ganeti use Haskell? What are Ganeti’s plans with respect to increasing or decreasing the ratio of Haskell code? ##

Ganeti first used Haskell as an experiment when we needed a compiled language that could perform fast computations on cluster metadata. Eventually, we found that Haskell had advantages in its type safety that made it easy for us to develop better code with fewer common errors without having to write extensive “type-specific” unit tests. At this point, we decided to expand our use of Haskell.

As of January 2014, we plan to convert only “infrastructural” code (job queue, locking) that was already very complicated in Python to Haskell. We’ll continue to use Python for Logical Units, backend RPCs, and hypervisor, storage, and networking implementations.

There are two main reasons why Haskell constitutes a low barrier:
  * Haskell doesn’t have any runtime dependency on the target systems.
  * For setup, Haskell only requires a development environment.
For good examples of setup, see the Ganeti documentation (in particular, [Haskell requirements](http://docs.ganeti.org/ganeti/master/html/install-quick.html#haskell-requirements)) and codebase (in particular, [devel/build\_chroot](http://git.ganeti.org/?p=ganeti.git;a=blob;f=devel/build_chroot;h=39bec386535dbf16283af592792ea90b732a794c;hb=HEAD)).

If some part of the Haskell codebase is too inflexible for your use, the Ganeti team is happy to help you understand how to customize the codebase, or how to make the codebase more modular and scriptable, if possible.