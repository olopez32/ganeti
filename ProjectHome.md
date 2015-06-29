# Ganeti Overview #

Ganeti is a cluster virtual server management software tool built on top of existing virtualization technologies such as [Xen](http://www.xen.org/) or [KVM](http://kvm.et.redhat.com/page/Main_Page) and other open source software.

Ganeti requires pre-installed virtualization software on your servers in order to
function. Once installed, the tool assumes management of the virtual instances (Xen DomU). Ganeti controls:
  * Disk creation management
  * Operating system installation for instances (in co-operation with OS-specific install scripts)
  * Startup, shutdown, and failover between physical systems
Ganeti is designed to facilitate cluster management of virtual servers and to provide fast and simple recovery after physical failures using commodity hardware.

Primary Ganeti web page is http://www.ganeti.org/.


# Features #

Ganeti provides the following features for managed instances:
  * Support for Xen virtualization:
    * Support for PVM and HVM instances
    * Support for live migration
    * Virtual console (on PVM) or VNC (on HVM) to control instances
    * Support for virtio or emulated devices
  * Support for KVM virtualization:
    * Support for live migration
    * Support for fully virtualized instances
    * Support for semi-virtualized instances (kernel residing on the host)
    * Support for VNC or serial access
    * Support for virtio or emulated devices
  * Cluster size of 1-150 physical nodes (recommended)
  * Disk management:
    * Plain LVM volumes
    * Files
    * Across-the-network RAID1 (using [DRBD](http://www.drbd.org/)) for quick recovery in case of physical system failure
  * Instance disk partitioning
  * Export/import mechanism for backup purposes or migration between clusters
  * Automated instance migration across clusters

# Documentation #

  * http://docs.ganeti.org
    * [Top-level documentation](http://docs.ganeti.org/ganeti/current/html/)
    * [Installation tutorial](http://docs.ganeti.org/ganeti/current/html/install.html)
    * [Man pages](http://docs.ganeti.org/ganeti/current/man/)
  * See http://docs.ganeti.org/ for documentation of older versions.

# Contact Information #

  * User discussions: [ganeti@googlegroups.com](http://groups.google.com/group/ganeti)
  * Ganeti development: [ganeti-devel@googlegroups.com](http://groups.google.com/group/ganeti-devel)
  * IRC: #ganeti on Freenode