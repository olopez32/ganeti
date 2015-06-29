<h1> Introduction </h1>

This document explains how you can easily test and try out Ganeti without needing a lot of extra hardware. This also provides a way for you to try out Ganeti on various platforms more easily.

See [GitHub](https://github.com/ramereth/vagrant-ganeti) for the most up to date documentation.

<h2>Table of contents</h2>


# Requirements #

  * VirtualBox >=4.1.x
  * Vagrant >=1.0.3

# Setup #

1. Install VirtualBox by going to their [download page](https://www.virtualbox.org/wiki/Downloads).

2. Install Vagrant
```
  gem install vagrant
```

3. Clone repository
```
  git clone git://github.com/ramereth/vagrant-ganeti.git
```

4. Initialize submodule(s)
```
  git submodule update --init
```

# Using Vagrant #

The Vagrantfile is setup to where you can deploy one, two, or three nodes depending on your use case. `Node1` will have Ganeti already initialized while the other two will only have Ganeti installed and primed.

For more information on how to use Vagrant, please [check out their site](http://vagrantup.com/docs/index.html).

## Starting a single node ##
```
    vagrant up node1
    vagrant ssh node1
```

## Starting node2 ##

NOTE: Root password is 'vagrant'.
```
    vagrant up node2
    vagrant ssh node1
    gnt-node add -s 33.33.34.12 node2
```

## Starting node3 ##

NOTE: Root password is 'vagrant'.
```
    vagrant up node3
    vagrant ssh node1
    gnt-node add -s 33.33.34.13 node3
```

# Accessing the nodes #

Add the following to your `/etc/hosts` files for easier access locally.
```
    33.33.33.10 ganeti.example.org
    33.33.33.11 node1.example.org
    33.33.33.12 node2.example.org
    33.33.33.13 node3.example.org
```

All the nodes are using `hostonly` networking with the following IP's:

  * ganeti.example.org (cluster IP) = 33.33.33.10
  * node1.example.org = 33.33.33.11
  * node2.example.org = 33.33.33.12
  * node3.example.org = 33.33.33.13

Additionally, I have setup several VM DNS names in the `/etc/hosts` of each node that you can use:

  * instance1.example.org
  * instance2.example.org
  * instance3.example.org
  * instance4.example.org

# RAPI Access #

The RAPI user setup for use on the cluster uses the following credentials.

  * user: vagrant
  * pass: vagrant

# Running different Ganeti versions #

This repo has been setup to deal with a variety of Ganeti versions for testing. Currently it only supports 2.4.X, 2.5.X, 2.6.x and any git tagged releases. To switch between the versions do the following:

  1. edit `modules/ganeti_tutorial/node{1-3}.pp`
  1. if using git, change `git` to `true`
  1. change `ganeti_version` to desired version
  1. redeploy the vm(s) (destroy, up)

# Node Operating System #

By default we use Ubuntu 11.10 for our node OS but we do have support for the following operating systems. Just run the vagrant commands from inside the appropriate folder.

  * Debian 6 (debian-6)
  * Debian 7 (debian-7)
  * CentOS 6 (centos-6)
  * CentOS 5 (work in progress)
  * Ubuntu 12.10 (work in progress)