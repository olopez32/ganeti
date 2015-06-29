# Background #

From 2.11 onwards, Ganeti has a new way of handling SSL certificiates. The details of this can be read in the design doc [Ganeti Node Security](http://git.ganeti.org/?p=ganeti.git;a=blob;f=doc/design-node-security.rst;h=05d28b001ae1407850c837d833a34510d2d57623;hb=HEAD).

To summarize, instead of only one SSL certificate "server.pem", we added individual client SSL certificiates "client.pem" to each node. A list of client certificate fingerprints of master candidate nodes is maintained in the configuration and distributed to all nodes as "ssconf\_master\_candidate\_certs" file.

However, this has turned out to have more problems than we expected. There a couple of known bugs which might render your cluster uncommunicative. This document shows you some hints and quick fixes for some of those situations.

# CN-encodings #

[Issue 1094](https://code.google.com/p/ganeti/issues/detail?id=1094) reports a problem which can cut off SSL communication between Ganeti clusters if different openssl/gnutls versions are running inside a cluster or if the server.pem is very old so that it was created with a different openssl/gnutls version than the new client.pem.

If you experience problems with nodes being unreachable by RPC, you might be affected by this. In particular in clusters with inhomogenic SSL (openssl/gnutls) versions, this is likely to happen after a master failover.

We are working on a fix, but up till then, you might need to use the following trick to run a 2.12 cluster without client.pem

# Ensure state of new SSL setup #

The new handling of SSL certificates in Ganeti has still some weaknesses. If you discover communication errors between your nodes on your cluster at version 2.12 or later, please check that the following things are correct:

  * All nodes have the **same** `/var/lib/ganeti/server.pem` file.
  * All nodes have a `/var/lib/ganeti/client.pem` file, which should be **different** for each node.
  * All nodes have a `/var/lib/ganeti/ssconf_master_candidate_certs` file.

### Fixes ###

  * If only the `server.pem` file is missing / inconsistent around nodes, just copy the file from the master node to all other nodes. Restart Ganeti on all nodes to make sure the daemons pick up the new certificate.
  * If the `client.pem` and/or the `ssconf_master_certificates_certs` is missing only on one or very few nodes (but not on the master node), your best bet is to re-add those nodes as currently there is no way to renew the certificates for one node only. (There is an [issue](https://code.google.com/p/ganeti/issues/detail?id=1043) for it.)
  * If the `client.pem` and/or the `ssconf_master_certificates_certs` is missing on the master node or one a lot of nodes in the cluster, your best bet is to follow the instruction in the next Section "Run 2.12 without client certificates" and **afterwards** run `gnt-cluster-renew-crypto --new-node-certificates`. The latter will then create new `client.pem` and `ssconf_master_candidate_certs`.


# Run 2.12 without client certificates #

It is possible to temporarily run a 2.12 cluster without client.pem certificates. This can be used as a last resort if you get hit by a bug.

However, note that this is not tested well, because it was only meant as a temporary state of a cluster. If you run a cluster this way, you should closely monitor operations like adding nodes, promoting/demoting nodes to/from master candidate, and master failovers.

To run a 2.12 cluster with the SSL setup of a pre-2.12 cluster (without client certificates), you need to do the following:

  1. Shutdown the Ganeti daemons on all nodes (e.g. `/etc/init.d/ganeti stop`.
  1. Remove (or move away) the two files `/var/lib/ganeti/client.pem` and `/var/lib/ganeti/ssconf_master_candidate_certs` on all nodes. It is important that this is done consistently across the cluster.
  1. Restart all Ganeti daemons on all nodes again (e.g. `/etc/init.d/ganeti start`)
  1. Run `gnt-cluster verify` to make sure everything is fine again. You will get a nagging message that you should run `gnt-cluster renew-crypto --new-node-certificates`. You need to ignore this as long as you want to run the cluster in pre-2.12 state.