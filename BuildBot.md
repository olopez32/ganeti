<h1> Introduction </h1>

We run a continuous build of the Ganeti unittests and vcluster qa using [buildbot](http://trac.buildbot.net/). The official [Ganeti buildbot](http://buildbot.ganeti.org) is running on [~okeanos](http://okeanos.grnet.gr),powered by [Synnefo](http://www.synnefo.org/) and Ganeti! We kindly thank grnet for their support.

Link to the web interface: http://buildbot.ganeti.org/ganeti/tgrid?length=25

<h2>Table of contents</h2>


# Overview #

As usual in buildbot, there are two parts:

  * the master machine, publicly accessible over HTTP
  * the slaves, which only need to talk to the master

Setup of the master machine is straightforward (it only needs to run
buildbot), with the note that the global buildbot configuration is
non-trivial.

The setup of the slaves, however, is more complex. Each slave needs to
be able to build Ganeti, with all the required dependencies, which are
“non-trivial” so to say.

Currently we have the following slaves defined:

  * debian wheezy 64bit
  * debian jessie 64bit
  * ubuntu 13.04 64bit
  * fedora 18 64bit

Feel free to prepare a new slave and ask us to add it to buildbot! The
only requirement is to be able to build Ganeti with all or most of its
dependencies (we can make exceptions if needed).

# Virtual cluster (VCluster) QA #

In addition to just build Ganeti and run unit tests, there is also a QA
build on a virtual cluster defined on buildbot. The rough overview of the process is:

  * A buildslave builds Ganeti (the Fedora 18 slave is used currently)
  * It uploads Ganeti to the QA machine
  * A large number of Ganeti commands are executed by the buildslave via `ssh` on the QA machine
    * A cluster is initialized
    * Virtual nodes (additional node daemons running on the QA machine) are added
    * Diskless instances for the fake hypervisor are created (but not actually run)

There are a number of requirements for this setup to work:
  * All runtime dependencies of Ganeti have to be installed on the QA machine
  * The buildslave has to have SSH access to the QA machine
    * In particular, the `buildbot` user on the buildslave has to be able to connect as `root` without password to the QA machine
  * SELinux interferes badly with the SSH based tests. That's why it's disabled on the buildslave and the QA machine

Currently we have the following QA machines defined:

  * fedora 18 64bit (the driving buildslave is also running fedora 18 64bit)

See also the [detailed description on how to setup a vcluster](Vcluster.md).

# KVM cluster QA #

Appart from the virtual cluster, we also have a QA on a real cluster. The cluster
is formed of 3 debian wheezy. These machines use kvm as hypervisor and a private
network on `eth1` as replication network. The master IP and the instance IPs also
all live the 192.168.0/24 on `eth1`.

The QA is run from the wheezy buildslave (snf-14476.vm.okeanos.grnet.gr).

# Buildbot machines #

  * master: buildbot.ganeti.org or snf-13819.vm.okeanos.grnet.gr
  * wheezy: snf-14476.vm.okeanos.grnet.gr
  * jessie: snf-472938.vm.okeanos.grnet.gr
  * ubuntu 13.04: snf-68991.vm.okeanos.grnet.gr
  * fedora 18: snf-69083.vm.okeanos.grnet.gr
  * QA-VCluster, fedora 18: snf-69749.vm.okeanos.grnet.gr
  * QA KVM-Cluster, debian wheezy: snf-192771.vm.okeanos.grnet.gr, snf-192805.vm.okeanos.grnet.gr, snf-192809.vm.okeanos.grnet.gr

# Buildmaster users #

The users (which can cancel builds, force builds, etc.) are managed by
hand (as we shouldn't need any user beside a generic one), in
`/srv/buildbot/masters/ganeti/htpasswd`. Note that htpasswd (from
apache2-utils) needs to be run with `-d`, as buildbot 0.8.6p1 doesn't
yet support md5/sha digests.

# Slave setup #

While the slaves are setup by slack, not everything was automated.

Initial setup means just installing slack, and pointing
`/etc/slack.conf` to localhost (assuming you ssh into the machines
with an rsync server running on your machine and exported to the slave on the correct port through the -R parameter of ssh):

```
…
SOURCE=rsync://localhost/slack
…
```

Then afterwards running slack should be enough.

Note that for tests involving exclusive storage (`exclusive-storage-instance-tests` and `cluster-exclusive-storage` options in the QA configuration) you need to have more than one LVM physical volume in the default volume group.

## Debian test build slaves ##

On grnet, the base images come with NetManager and a number of other
daemons that are useful for desktop use; as the VMs don't have lots of
memory, I've disabled them and switched back to plain
`/etc/network/interfaces`.

On wheezy, pyinotify throws epydoc into a fit, so the solution (urgh!)
is to hand-modify `/usr/share/pyshared/pyinotify.py` and remove the
`class _PyinotifyLogger(logging.getLoggerClass())` definition (this
was removed as well in an upstream commit,
`98c5f41a6e2e90827a63ff1b878596f4080481cc`).

## Shelltestrunner on squeeze ##

The `shelltestrunner` cabal package is very hard to install on
squeeze, as it depends (for some purely cosmetical things) on newer
packages that can be installed on ghc 6.12. So the installation of
shelltestrunner was done manually:

```
cabal unpack shelltestrunner
```

And apply the following patch:

```
diff -ur old/shelltestrunner-1.3.1/shelltest.hs new/shelltestrunner-1.3.1/shelltest.hs
--- xc/shelltestrunner-1.3.1/shelltest.hs       2013-01-28 14:03:38.000000000 +0200
+++ shelltestrunner-1.3.1/shelltest.hs  2013-01-22 18:16:55.000000000 +0200
@@ -34,7 +34,6 @@
 import qualified System.FilePath.Find as Find (extension)
 import Control.Applicative ((<$>))
 import Data.Algorithm.Diff
-import Distribution.PackageDescription.TH (packageVariable, package, pkgVersion)

 import PlatformString (fromPlatformString, toPlatformString)

@@ -43,7 +42,7 @@

 progname, version, progversion :: String
 progname = "shelltest"
-version = $(packageVariable (pkgVersion . package))
+version = "1.3.1"
 progversion = progname ++ " " ++ version
 proghelpsuffix :: [String]
 proghelpsuffix = [
diff -ur old/shelltestrunner-1.3.1/shelltestrunner.cabal new/shelltestrunner-1.3.1/shelltestrunner.cabal
--- xc/shelltestrunner-1.3.1/shelltestrunner.cabal      2013-01-28 14:03:38.000000000 +0200
+++ shelltestrunner-1.3.1/shelltestrunner.cabal 2013-01-22 18:06:25.000000000 +0200
@@ -29,7 +29,6 @@
   ghc-options:    -threaded -W -fwarn-tabs
   build-depends:
                  base                 >= 4     && < 5
-                ,cabal-file-th
                 ,filemanip            >= 0.3   && < 0.4
                 ,HUnit                            < 1.3
                 ,cmdargs              >= 0.7   && < 0.11
```

After that, running `cabal install --global` as root should do the right thing.

## Starting/stopping ##

The buildmaster setup is automatically started, as the buildbot is
installed from Debian packages. Manually starting/stopping/reconfiguring is possible via:

```
cd /srv/buildbot/masters/ganeti
buildbot stop|start|checkconfig|reconfig
tail -f twistd.log
```

The wheezy slave is started/stopped as well (for the same reason,
automatically from packages), but the squeeze slave is only started at
boot time from `/etc/rc.local`. All slaves are located under
`/srv/buildbot/slaves`, so starting/stopping them is a matter of:

```
cd /srv/buildbot/slaves/unittests-wheezy64
buildslave stop|start
```

# Source code #

The source code is stored in Git under git.ganeti.org, in the
[buildbot repository](http://git.ganeti.org/buildbot.git/).