<h1> Introduction </h1>

Ganeti is applying to be a hosting organization for GSoC 2013. This document outlines our ideas.

<h2>Table of contents</h2>


# Better openvswitch Ganeti support #

```
Objectives:
- support for openvswitch features on the instance NICs (dynamic vlan configuration, bandwidth control)
- openvswitch clustering and tunneling: self configuration, system level networks

Benefits:
- update Ganeti to support more scalable networking
- make it easy to administer an openvswitch installation inside Ganeti
- allow inter-vm communication (when requested) without explicit support in the switches config

Requirements:
- Interest in distributed systems and networking
- Knowledge of Python and Haskell
- Ability to write high quality code, tests and QA
- Ability to interact with Ganeti and Openvswitch developers
- Access to multiple machines (or VMs) to test the project and run QA
```

# Dynamic networking #

```
Objectives:
- possibility to use a network interface name instead of an IP address for VNC/spice bind addresses, and for secondary ip
- network interfaces (bridges, routes, etc) dynamically configured and unconfigured on nodes as needed by instances

Benefits:
- increase scalability and ease of configuration
- make network configuration more customizable
- make network configuration easier to maintain

Requirements:
- Interest in distributed systems and networking
- Knowledge of Python and Haskell
- Ability to write high quality code, tests and QA
- Ability to interact with Ganeti developers and other summer of code students (may interact with the idea above)
- Access to multiple machines (or VMs) to test the project and run QA
```

# VDE Ganeti support #

```
Objectives:
- support for [http://wiki.virtualsquare.org/wiki/index.php/VDE_Basic_Networking VDE] as a possible instance nic
- automatically configure a VDE network between Ganeti nodes for instances
- support kvm instances with VDE userspace network backends

Benefits:
- reduce the number of privileges Ganeti requires to run (opening the space to run as non-root)
- make it easy to test new network configurations in userspace

Requirements:
- Interest in distributed systems and networking
- Knowledge of Python and Haskell
- Ability to write high quality code, tests and QA
- Ability to interact with Ganeti developers and other summer of code students (may interact with the ideas above)
- Access to multiple machines (or VMs) to test the project and run QA

Notes:
- Probably should be staffed after the two ideas above
```




# Better RADOS/Ceph Ganeti support #

```
Objectives: 
- support for userspace mode for RADOS+KVM
- support for running RADOS on Ganeti nodes (self-configuring of Ganeti nodes as a RADOS backend)

Benefits:
- make it easier to deploy a RADOS infrastructure with Ganeti
- support the better performance and higher resilience of running RADOS in userspace

Requirements:
- Interest in distributed systems and storage
- Knowledge of Python and Haskell
- Ability to write high quality code, tests and QA
- Ability to interact with Ganeti developers and RADOS developers
- Access to multiple machines (or VMs) to test the project and run QA
```


# GlusterFS Ganeti support #
```
Objectives:
- add Gluster as a storage backend
- add support for userspace gluster in kvm
- support for running gluster on ganeti nodes (self-configuring of Ganeti nodes as a gluster backend)

Benefits:
- allow users to deploy Ganeti clusters with Gluster
- bring Gluster up to speed w.r.t. RADOS and DRBD in support

Requirements:
- Interest in distributed systems and storage
- Knowledge of Python and Haskell
- Ability to write high quality code, tests and QA
- Ability to interact with Ganeti developers and RADOS developers
- Access to multiple machines (or VMs) to test the project and run QA
```

# KVM resource constrainment #
```
Objectives:
- cgroups integration for more reliable resource constrainment
- linux containers integration for better control of what the qemu executable can access

Benefits:
- better segregation in case of virtualization/qemu breakup
- better resource accounting

Requirements:
- Interest in virtualization and security
- Knowledge of Haskell and Python
- Ability to write high quality code, tests and QA
- Ability to interact with Ganeti developers
- Access to multiple kvm-enabled machines (or VMs) to test the project and run QA
```

# KVM: Hugepages support #
```
Objectives:
- design a way to support different "memory pools" and fix KVM free memory reporting
- handle node hugepage memory correctly in memory reporting
- allow to create/start instances on nodes with hugepages without failing for non-hugepages memory accounting
- handle the hugepages pool in iallocator/hbal
- run benchmarks and report performance of normal instance VS hugepages instance VS THP instance

Benefits:
- better separation of instance memory and OS memory for KVM
- improved performance for hugepages instances

Requirements:
- Interest in virtualization and security
- Knowledge of Haskell and Python
- Ability to write high quality code, tests and QA
- Ability to interact with Ganeti developers
- Access to multiple kvm-enabled machines to test the project and run QA
```


# Improve linux-HA integration #
```
Objectives:
- allow ganeti to automatically set up linux-HA on the cluster

Benefits:
- remove the need for hand-configuration, and make it easy to deploy Ganeti+Linux-HA together
- add QA of the current linux-HA resources for Ganeti
- fully enable self-repairing clusters

Requirements:
- Interest in virtualization and security
- Knowledge of Haskell and Python
- Ability to write high quality code, tests and QA
- Ability to interact with Ganeti developers
- Access to multiple machines in order to thoroughly test the project
```

# Add Ganeti provider to Vagrant #
```
Objectives:
- allow vagrant to communicate and interact with a Ganeti cluster

Benefits:
- Allow vagrant users to power their virtual machines via a private Ganeti cloud
- Gives vagrant users a testing environment that could scale into the hundreds vms
- Provides a unified and popular remote command line interface to Ganeti

Requirements:
- Write a vagrant provider that connects to the Ganeti RAPI which can deploy/connect to a virtual machine
- Knowledge in Ruby
- Ability to write high quality code, tests and QA
- Ability to interact with Ganeti developers
```