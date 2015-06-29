<h1> Introduction </h1>

FAI is a very flexible and pluggable installation framework.
Apart from being used for network installations in large infrastructures and for building custom installation CD's, it has a "dirinstall" to create a changeroot.

This can be a ganeti disk, too.

Another point for FAI's flexibility: FAI can be used to install systems with many distributions, even a proof of concept for Windows exists.

<h2>Table of contents</h2>


# Setting up a Ganeti OS template for FAI #

Basically, it's just a short patch to run against the "create" script of the default Debian OS template coming with Ganeti.

Just extract the default debian template, copy the full driectory(e.g. to the name fai-os-template) and replace the file "create" with this code:

```
#!/bin/sh

# Copyright (C) 2007 Google Inc., Henning Sprang - Silpion IT Solutions GmbH
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 2 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful, but
# WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
# General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA
# 02110-1301, USA.

set -e

TEMP=`getopt -o i:b:s: -n '$0' -- "$@"`

if [ $? != 0 ] ; then echo "Terminating..." >&2 ; exit 1 ; fi

# Note the quotes around `$TEMP': they are essential!
eval set -- "$TEMP"

while true; do 
	case "$1" in
		-i) instance=$2; shift 2;;

		-b) blockdev=$2; shift 2;;

		-s) swapdev=$2; shift 2;;

		--) shift; break;;

		*)  echo "Internal error!"; exit 1;;
	esac
done
if [ -z "$instance" -o -z "$blockdev" -o -z "$swapdev" ]; then
	echo "Missing -i or -b or -s argument!"
	exit 1
fi

mkswap $swapdev
mke2fs -Fjq $blockdev
TMPDIR=`mktemp -d` || exit 1
trap "umount $TMPDIR; rmdir $TMPDIR" EXIT
mount $blockdev $TMPDIR

fai -N  -v -u $instance dirinstall $TMPDIR

# TODO: should go into FAI - XENU-DYNAMIC-NET
rm -f "$TMPDIR/etc/udev/rules.d/z25_persistent-net.rules"


umount $TMPDIR
rmdir $TMPDIR
trap - EXIT

exit 0
```


# Configuring FAI #
Sure, [FAI](http://www.informatik.uni-koeln.de/fai/) should be installed first before being configured. On Debian and Ubuntu, simply install the package fai-quickstart. On other distributions, it's not really tested, just by preference of the developers. But, on the other hand, it should be fairly easy to install, especially if you only want to use the dirinstall feature. If you really find it interesting, the FAI Team will support you to get it running on your favorite distribution.

FAI will do the things you configure it to do. Mainly, classes are configured based on things like a hostname (or many other things), and for each class you can define
  * Which software packages are needed
  * Which Which configurations they need (based on files to be copied, or edited from defaults)

Giving a full Tutorial about FAI is out of the scope of this document - see http://faiwiki.debian.net for details, and [here](http://faiwiki.debian.net/images/7/74/Fai_presentation_extremadura2007.pdf) is a presentation with an introduction.