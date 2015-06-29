# Introduction #

This is a list of all known Ganeti ExtStorage providers.

You may find some of them included in the Ganeti source code under `examples/extstorage/`. If you have a working ExtStorage provider and would like to share it with the Ganeti community, please send us an email on the Ganeti public list, along with a link to your provider, and we will include it in this list.

# ExtStorage interface #

ExtStorage providers plug into Ganeti's ExtStorage interface and allow you to integrate Ganeti with external shared storage, most commonly SAN appliances. To learn more about the ExtStorage interface, see the following links:
  * [Ganeti ExtStorage Interface slides from GanetiCon2013](https://docs.google.com/a/google.com/file/d/0B934VF_cTqnwc0NGcEx4OXl3NDA/edit)
  * [Ganeti ExtStorage Interface man page](http://docs.ganeti.org/ganeti/master/html/man-ganeti-extstorage-interface.html)
  * [Ganeti ExtStorage Interface design doc](http://docs.ganeti.org/ganeti/master/html/design-shared-storage.html#introduction-of-the-external-storage-interface)

To read an example of how to install and configure an ExtStorage provider see this [Google Groups email thread](https://groups.google.com/forum/?fromgroups=#!searchin/ganeti-devel/shared-filer/ganeti-devel/kGTje2FP94k/in8s6RVFQqQJ), explaining how to set up the ExtStorage interface.

# ExtStorage providers list #

## Shared Filer ##

This is the sample provider submitted along with the ExtStorage Interface, to act as the example on how users can implement their own ExtStorage providers. It does not interact with a SAN device; it handles VM disks as plain files under a shared directory, which is configurable via the `shared_dir` ext-param (to showcase how ext-params work).

Essentially, it implements Ganeti's native `-t sharedfile` suppport, over the ExtStorage Interface.
To learn more on how to set it up, see the corresponding [Google Groups email thread](https://groups.google.com/forum/?fromgroups=#!searchin/ganeti-devel/shared-filer/ganeti-devel/kGTje2FP94k/in8s6RVFQqQJ).

Provider's link: https://code.grnet.gr/projects/extstorage/repository/revisions/master/show/shared-filer

## RBD ##

This is a second sample provider that implements Ganeti's native RBD support (`-t rbd`), over the ExtStorage interface. It is meant to act as an example of a more complex provider. The recommended way to integrate Ganeti with Ceph's RADOS is via the native RBD support and not this provider.

Provider's link: https://code.grnet.gr/projects/extstorage/repository/revisions/master/show/rbd

## IBM Storwize Family ##

This provider integrates Ganeti with the IBM Storwize product family. It has been tested to work with IBM Storwize v7000.

Provider's link: https://code.grnet.gr/projects/extstorage/repository/revisions/master/show/svc

## HP EVA Storage ##

This provider integrates Ganeti with the HP EVA Storage Array.

Provider's link: https://github.com/DSI-Universite-Rennes2/hpeva

## ZFS ##

This provider integrates Ganeti with [ZFS](http://en.wikipedia.org/wiki/ZFS).

Provider's link: https://github.com/ffzg/ganeti-extstorage-zfs

## cLVM and "Shared" LVM ##

This provider allows Ganeti to use clustered LVM or LVM volumes on shared storage.

Provider's link: http://www.goodbytez.de/ganeti/README