<h1> Introduction: Ganeti in VirtualBox </h1>

Here's a method for getting a ganeti environment set up in VirtualBox on a workstation. This procedure isn't difficult, but there are a lot of steps. I've made a few choices that may or may not suit you:
  1. I'm using dnsmasq to handle internal dhcp and dns.
  1. I've allocated 768M ram per virtual node for 2G ram dedicated to VirtualBox. You can probably get away with less ram per node, like say 512M. Don't be afraid to get better hardware to make your life much much easier.
  1. I'm using VirtualBox host-only networking to create the required networks.
  1. I'm using debian stable (squeeze) as the operating system for the nodes.

<h2>Table of contents</h2>


# Setting up the host #

First, install the required packages:
```
sudo aptitude install virtualbox-ose dnsmasq clusterssh
```

Download debian stable ISO.

Select what netblocks you want to use, non-routable is recommended. I chose to use 192.168.10.0/24 for the normal netblocks, and 192.168.20.0/24 for the replication netblock.

# Setting up VirtualBox #

## Create virtual networks ##
In File->Preferences->Network, create two host-only networks. Configure them as follows:
```
vboxnet0:
IPv4 Address: 192.168.10.254
IPv4 Network Mask: 255.255.255.0
DHCP Server: Disabled

vboxnet1:
IPv4 Address: 192.168.20.254
IPv4 Network Mask: 255.255.255.0
DHCP Server: Disabled
```

## Create nodes ##
Choose where to put your disk images under File->Settings->General->Default Hard Disk Folder.

For each desired node:
  1. Create a new virtual machine in VirtualBox
  1. Set the OS to debian 64bit
  1. Give it 768M of ram, 8G sparse hard drive named `<node>`.
  1. Set first nic to Host-only Adapter, Name: vboxnet0, Type: PCnet-FAST III, Promiscuous Mode: Allow All.
  1. Set second nic to Host-only Adapter, Name vboxnet1, Type PCnet-FAST III, Promiscuous Mode: Allow All.
  1. Add another 100G sparse hard drive (this will be used for xenvg) named `<node>vg`.
  1. Mount debian stable ISO in virtual cd drive.

Add hosts to /etc/hosts:
```
# Nodes
192.168.10.1 node1
192.168.10.2 node2
192.168.10.3 node3

# Clusters
192.168.10.4 cluster1

# Replication network
192.168.20.1 node1rep
192.168.20.2 node2rep
192.168.20.3 node3rep

# Instances
192.168.10.101 inst1
192.168.10.102 inst2
192.168.10.103 inst3
```

Configure dnsmasq by editing /etc/dnsmasq.conf, replacing all "`<MAC>`" with the MAC addresses of the virtual NICs, in the format 00:00:00:00:00.
```
interface=vboxnet0,vboxnet1

dhcp-range=vboxnet0,192.168.10.1,192.168.10.253,12h
dhcp-range=vboxnet1,192.168.20.1,192.168.20.253,12h

dhcp-host=<MAC>,node1,1d
dhcp-host=<MAC>,node2,1d
dhcp-host=<MAC>,node3,1d

dhcp-host=<MAC>,node1rep,1d
dhcp-host=<MAC>,node2rep,1d
dhcp-host=<MAC>,node3rep,1d

# Don't hand out default route on replication network
dhcp-option = net:vboxnet1, option:router
```

Add to /etc/rc.local:
```
iptables -F FORWARD
iptables -A FORWARD -i vboxnet0 -j ACCEPT
iptables -t nat -F POSTROUTING
iptables -t nat -A POSTROUTING -s 192.168.10.0/24 -j MASQUERADE
```

Enable forwarding in /etc/sysctl.conf:
```
net.ipv4.ip_forward=1
```

# Setting up node #

For each node:
  * Boot machine off debian ISO, install debian stable.
  * Setup remote administration:
```
aptitude install openssh-server
```
  * Set up ssh keys so you can ssh in as root without password.

Now, switch to using clusterssh to work on all nodes at once:
```
cssh root@node{1..3}
```

  * Setup ntp:
> Install ntp:
```
aptitude install ntp ntpdate
```
> Edit /etc/ntp.conf, replace all server lines with:
```
server 192.168.10.254 iburst
```
> Do initial sync and restart ntpd:
```
invoke-rc.d ntp stop
ntpdate 192.168.10.254
invoke-rc.d ntp start
```

  * Install xen following directions on debian wiki: [Xen installation for debian squeeze](http://wiki.debian.org/Xen#Installation_on_squeeze). For reference here are the necessary steps:
```
# Install xen packages:
aptitude install xen-hypervisor-4.0-amd64 linux-image-xen-amd64

# Fix grub2's xen handling
mv -i /etc/grub.d/10_linux /etc/grub.d/50_linux
echo "" >> /etc/default/grub
echo "# Disable OS prober to prevent virtual machines on logical volumes from appearing in the boot menu." >> /etc/default/grub
echo "GRUB_DISABLE_OS_PROBER=true" >> /etc/default/grub
update-grub2
```

  * Check to see what packages ganeti2 in experimental would pull in, but do not install the package:
```
aptitude -t experimental install ganeti2
```
  * Install all the deps for ganeti2 (plus a few extras that we need):
```
aptitude install dbus debootstrap drbd8-utils dump
ganeti-instance-debootstrap iputils-arping javascript-common kpartx libaio1 \
libasound2 libasyncns0 libbluetooth3 libbrlapi0.5 libcurl3-gnutls libdbus-1-3 \
libdirectfb-1.2-9 libflac8 libjs-jquery libogg0 libpulse0 libsdl1.2debian \
libsdl1.2debian-alsa libsndfile1 libsvga1 libsysfs2 libts-0.0-0 libvdeplug2 \
libvorbis0a libvorbisenc2 libx86-1 libxi6 libxtst6 lvm2 python-openssl \
python-pyinotify python-pyparsing python-simplejson qemu-kvm socat tsconf \
wwwconfig-common x11-common rsync tcpdump python-pycurl python-paramiko
```

  * Follow the steps [here](http://docs.ganeti.org/ganeti/current/html/install.html#xen-settings), starting with "Xen settings" and stopping after "Configuring LVM", making the following modifications:

  1. We're using grub2, so skip the part about boot options and set the dom0 memory limit with:
```
echo 'GRUB_CMDLINE_XEN_DEFAULT="dom0_mem=256M"' >> /etc/default/grub
update-grub2
```
  1. drbd is already installed from the above aptitude command, but still needs configuration. This probably just means doing the following:
```
echo drbd minor_count=128 usermode_helper=/bin/true >> /etc/modules
```
  1. Network configuration is replaced by editing /etc/network/interfaces to be:
```
auto xen-br0
iface xen-br0 inet dhcp
    bridge_ports eth0
    bridge_stp off
    bridge_fd 0

auto eth1
iface eth1 inet dhcp
```
  1. The lvm device is /dev/sdb

  * Reboot into the xen dom0.

# Setting up ganeti #

Distribute ganeti tarball:
```
make dist
for host in node{1..3}; do
  scp ganeti-*.tar.gz root@$host:
done
```

Install ganeti on all nodes:
```
./configure --prefix=/usr --sysconfdir=/etc --localstatedir=/var \
--with-os-search-path=/srv/ganeti/os,/usr/share/ganeti/os
make && make install
```

For one node in each cluster (node1 in this example):
  * Initalise cluster:
```
gnt-cluster init -s 192.168.20.1 cluster1
```
  * Add other nodes to cluster:
```
gnt-node add -s 192.168.20.2 node2
gnt-node add -s 192.168.20.3 node3
```
  * Configure xen parameters:
```
# Use initrd
gnt-cluster modify -H xen-pvm:initrd_path=/boot/initrd-2.6-xenU
# Use correct root device
gnt-cluster modify -H xen-pvm:root_path=/dev/xvda1
```
  * Now, you should be able to create an instance:
```
gnt-instance add -o debootstrap+default -s 500m -t drbd -n node1:node2 inst1
```
  * But if you want the instance to get the right IP address from dhcp, look up its MAC address in `gnt-instance info` and add a line to `/etc/dnsmasq.conf`:
```
dhcp-host=aa:00:00:fe:8a:7f,inst1,192.168.10.101
```