<h1> Introduction </h1>

ganeti-p2v-transfer is a tool for converting a physical computer into a ganeti instance. It consists of two parts, a ganeti instance OS template that allows the instance to be booted to receive the files, and a script that is run on the source machine to make the transfer.

<h2>Table of contents</h2>


# Requirements #

The following external programs are used by this package:

  * [paramiko](http://www.lag.net/paramiko/) is needed on the transfer OS (live image booted on the source machine) for making SSH connections
  * [pymox](http://code.google.com/p/pymox/) is required to run the unit tests (make check)
  * [rst2html, from docutils](http://docutils.sourceforge.net/) is needed to build the html documentation (make doc)

# Installation #
Determine location of OS template directory configured into ganeti.

> Short version: If ganeti was built from source, this is probably `/srv/ganeti/os` only. If ganeti was installed from a package, `/usr/share/ganeti/os` should be included, so the `--with-os-dir` parameter can be omitted below.

> Long version: Look for "OS Search Path" in the output of `gnt-cluster info`. If not there, look for a variable named OS\_SEARCH\_PATH in `_autoconf.py` in the installed ganeti files (e.g. `/usr/lib/python2.4/site-packages/ganeti/_autoconf.py`).

> Using the template directory location for `$OS_PATH`, distribute and install the package on the cluster:
```
gnt-cluster copyfile /root/ganeti-p2v-transfer-0.1.tar.gz
gnt-cluster command "tar xf ganeti-p2v-transfer-0.1.tar.gz &&
  cd ganeti-p2v-transfer-0.1 &&
  ./configure --prefix=/usr --localstatedir=/var \
    --sysconfdir=/etc \
    --with-os-dir=$OS_PATH &&
  make install-target"
```

Configure bootstrap OS

> Edit `/etc/ganeti/instance-p2v-target/p2v-target.conf` to uncomment the appropriate value of EXTRA\_PKGS. Depending on your setup, you may need to change other values as well. The file `doc/README.debootstrap.html` has more details.

> Copy the edited file to all nodes:
```
gnt-cluster copyfile /etc/ganeti/instance-p2v-target/p2v-target.conf
```

Generate bootstrap initrd

> The bootstrap OS must be started with a specialized initrd that moves all files into RAM before finishing the boot. Create this initrd by running:
```
sudo make_ramboot_initrd.py -V $DEB_KERNEL
```
> on the master node, where DEB\_KERNEL is the name of a kernel that can be used to boot a debootstrap instance. That is, if the filename of the kernel is /boot/vmlinuz-2.6.32-generic, DEB\_KERNEL would be "2.6.32-generic". This command should create the file /boot/initrd.img-$DEB\_KERNEL-ramboot. Copy this file to the other nodes with:
```
gnt-cluster copyfile /boot/initrd.img-$DEB_KERNEL-ramboot
```

Create login keypair

> Users will authenticate to their instances using an SSH keypair generated in advance by the administrator. The public key will be installed into root's .ssh/authorized\_keys file on the instance, and the private key will be provided to the user so that they can make the transfer. Generate the keys, with no passphrase, using the commands:
```
ssh-keygen -t dsa -N "" -f /etc/ganeti/instance-p2v-target/id_dsa
gnt-cluster copyfile /etc/ganeti/instance-p2v-target/id_dsa.pub
```
> Keep the private key (/etc/ganeti/instance-p2v-target/id\_dsa) somewhere safe, and give it to users who wish to use the P2V system.

# Workflow #

Administrator: create target instance

> Create an instance with the `p2v-target+default` OS and whatever parameters you need. The default kernel and initrd of the instance should be ones that are both compatible with and installed on the source OS. Also pass the `--no-start` flag, because we want to use the specially generated initrd for the boot rather than the default one.
```
gnt-instance add -t<template> -s<size> -o p2v-target+default \
-n<nodes> --no-start <hostname>
```
> Now boot the instance using the bootstrap kernel and initrd:
```
gnt-instance start -H kernel_path=/boot/vmlinuz-$DEB_KERNEL,\
initrd_path=/boot/initrd.img-$DEB_KERNEL-ramboot <hostname>
```

User: Start the transfer

> Before you begin, you will need the private key corresponding to the public key installed on the instance. Your administrator will provide this to you.

> Boot the source machine from a LiveCD or PXE image. Extract the ganeti-p2v-transfer tarball and run:
```
./configure -prefix=/usr --localstatedir=/var \
  --sysconfdir=/etc
sudo make install-source
```
> This will install the p2v\_transfer.py script. The script requires the following arguments:

  * root\_dev: the device file for the disk on which the root filesystem of the source machine is stored
  * target\_host: the hostname or IP address of the instance to receive the transfer
  * private\_key: the private key obtained from the administrator

> Run the script, and your data will be transferred:
```
sudo p2v_transfer.py $root_dev $target_host $private_key
```

> When the transfer finishes, the script will shut down the instance. When the ganeti watcher restarts it, log in and make sure that everything works.

# Troubleshooting #

## Bootstrap OS does not boot properly ##

These instructions suggest building the initrd on a node, for convenience. However, it is possible that there are incompatibilities between the initramfs-tools installed on the node and the kernel that will be used for the bootstrap OS. In this case, the bootstrap OS may not boot, or may not be able to find the root device. If this happens, a good way to improve compatibility is to use a machine that is already running the instance kernel, perhaps a "normal" (non-p2v) instance on the same cluster. Install and run `make_ramboot_initrd.py` on this machine to generate the desired initrd.

Another possibility is that the bootstrap OS does not have enough RAM to complete its boot. Since the bootstrap OS must be copied entirely into RAM, instances with small memory sizes are not supported. I have had good luck using 768MB of instance memory.

## No such script: /usr/share/debootstrap/scripts/squeeze ##

The version of debootstrap installed on the nodes may not be recent enough to support installing squeeze. Try changing the SUITE variable in `/etc/ganeti/instance-p2v-target/p2v-target.conf` to something older:
```
SUITE="lenny"
```