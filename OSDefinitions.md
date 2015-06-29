# Introduction #

This is a list of all known Ganeti OS definitions.

You may find some of them included in the Ganeti source code under `examples/os/`. If you have a working OS definition and would like to share it with the Ganeti community, please send us an email on the Ganeti public list, along with a link to your definition, and we will include it in this list.

# OS interface #

An OS definition plugs into Ganeti's OS interface and prepares the instance's newly provisioned disk, executing all the required customization operations needed, before Ganeti spawns the instance (with that disk as its first disk). To learn more about the OS interface see the following links:
  * [OS Interface man page](http://docs.ganeti.org/ganeti/master/man/ganeti-os-interface.html)
  * [OS installation redesign](http://docs.ganeti.org/ganeti/master/html/design-os.html)

# OS definitions list #

## ganeti-instance-debootstrap ##

This is the official sample OS definition provided by the Ganeti team.

Definition's link: http://git.ganeti.org/?p=instance-debootstrap.git;a=summary

## ganeti-instance-image ##

From the official site:
_Ganeti Instance Image is guest OS definition for Ganeti that uses either filesystem dumps or tar ball images to deploy instances. The goal of this OS definition is to allow fast and flexible installation of instances without the need for external tools such as debootstrap. It was originally based on ganeti-instance-debootstrap._

Definition's link: https://code.osuosl.org/projects/ganeti-image

## ganeti-os-defs ##

From the official site:
_In a few words this package contain 2 OS definition to handle fully (KVM and XEN) virtualized Windows and Linux machines:
  * linux-image -> definition to handle fully virtualized linux systems.
  * raw-image -> definition to handle fully virtualized machines (such as windows)._

Definition's link: http://sourceforge.net/p/ganeti-os-defs/home/Home/

## snf-image ##

From the official site:
_snf-image is a Ganeti OS definition. It allows Ganeti to launch instances from predefined or untrusted custom Images. The whole process of deploying an Image onto the block device, as provided by Ganeti, is done in complete isolation from the physical host, enhancing robustness and security.
snf-image supports KVM and Xen based Ganeti clusters._

snf-image is an advanced OS definition that runs all customization tasks (e.g., setting passwords, injecting files, setting hostnames) inside a virtualized environment (helper VM) for maximum security. It is designed, implemented and maintained by the [Synnefo](http://www.synnefo.org) team and officially supports the following guest OSes:
  * Linux: Debian, Ubuntu/Kubuntu, CentOS, Fedora, OpenSUSE
  * Windows: Windows 2008 [R2](https://code.google.com/p/ganeti/source/detail?r=2), Windows 2012
  * BSD: FreeBSD, NetBSD, OpenBSD

snf-image also supports all images created with the [snf-image-creator](http://www.synnefo.org/docs/snf-image-creator/latest/) tool.

Definition's link: http://www.synnefo.org/docs/snf-image/latest/