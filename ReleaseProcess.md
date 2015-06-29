#summary How to perform a new Ganeti release.
#labels Phase-Deploy

<h1> Introduction </h1>

This is the recommended procedure to perform a new Ganeti release.

<h2>Table of contents</h2>



# Create a new `stable-*` branch #

Releases are performed on stable branches (they are usually named stable-X.Y). In order to create a release, you must have the corresponding stable branch in place to prepare the release.  Before releasing, check that all the features and bug fixes that need to be released are present in the correct stable branch. If any features or bug fixes are missing, perform the appropriate merges or cherry-picks.

The steps to create this branch are as follows (we assume that `${BRANCH_VERSION}` holds the new version like `2.10`):

  1. Prepare and stabilize the master branch such that all tests and QA run through for the commit upon which you want to the branch to be based.
  1. Create the stable branch locally:
```
git checkout -b stable-${BRANCH_VERSION}
```
  1. Verify that the new branch points at the correct commit (the commit which passed tests and doclint):
```
git show --oneline -s stable-${BRANCH_VERSION}
```
  1. Have someone review the branch:
```
git push -u --dry-run origin stable-${BRANCH_VERSION}
```
  1. Push the branch with the previous command, but omitting `--dry-run`.
  1. Adapt buildbot configurations such that they also build and test the newly created branch. You most likely will need to touch the `master.cfg` and the `json` files of the upgrade tests, creating a new one for the new upgrade step. [Example patch](https://groups.google.com/forum/#!search/$20Adding$20new$20branch$202.11$20to$20buildbot/ganeti-devel/xjhxhvmcA8M/xh1zVZ2gKYIJ) Be aware of different buildbot installations!
  1. On master, bump version to the new development version and make sure that `cfgupgrade --downgrade` is a no-op. Note that you would need to add new fields to the new config file "cluster\_config-....json"depending on which were added in this version. It is unknown where to get this information from, running cfgupgrade on the old one will help or asking your beloved team mates. [Example patch](https://groups.google.com/forum/#!search/%22Prepare$20master$20branch$20for$202.11$20devel$20cycle%22/ganeti-devel/-YNt_nPr0fk/vYmA9jEN7uoJ)
  1. On the new stable branch, prepare the release as described below.


# Prerequisites #

  * Code conforms to the [Ganeti Python style guide](StyleGuide.md). (Adherence to the style guide should be checked during CL reviews.)
  * There are no high or critical priority bugs in the [Ganeti issue tracker](https://code.google.com/p/ganeti/issues/list). Filter for `Milestone=Release2.11 Type=Defect -status:Fixed`
  * All commits from the previous stable branch are merged into the version to be released.
  * Included documentation, readme files, and comments reflect the version to be released (for reference, see commits b830193c2, efe2137a, aff49a15a7).
    * Make sure that a doc/design-_version_.rst file exists (and is linked to from doc/index.rst), and that it contains all the design documents implemented or partially implemented in the current version. You must also remove the completely implemented designs  from doc/design-draft.rst. **Troubleshooting**: If you receive complaints about a file design file that is only linked in a doc/design-_version_.rst file which is not present in any TOC, add that file to ```doc/index.rst```, in the _hidden_ section.
    * Make sure that the content is up to date and change the version number in the following locations:
      * README
      * doc/design-draft.rst
      * doc/hooks.rst
      * doc/iallocator.rst
      * doc/security.rst
      * doc/virtual-cluster.rst
  * The NEWS file is updated.
  * Unittests and checks (`make commit-check`) from a pristine checkout do not fail:
```
URL=git://git.ganeti.org/ganeti.git devel/release origin/stable-x.y
```
  * You have run the test that requires root permissions. Doing so requires having the (optional) `fakeroot` dependency installed.
  * Burnin has been run for all hypervisor and disk types.
  * QA suite passes.
  * Before a release candidate, you made sure that:
    * The installation guide has been verified and tested. Ganeti must be able to be built with only the minimum required dependencies. You can test this, by setting up a (virtual) machine with a minimal debian (for example netinstall) installed. Then go through the `INSTALL` file, install only the packages described as mandatory and follow the installation instructions.
    * The upgrade/downgrade procedure has been tested. As the very least:
      1. Create an empty cluster with the previous stable version.
      1. Add an instance.
      1. Upgrade to the version to be released.
      1. Check that everything works as expected.
      1. Add a new instance.
      1. Downgrade to the previous version.
      1. Check that everything works.
  * You compared the .tar.gz obtained with `md5sum <tar_gz_package>./autogen.sh && ./configure && make dist` from a pristine checkout of the code to the .tar.gz of the previous release, and made sure that only expected modifications happened. The [Tardiff](http://tardiff.coolprojects.org/) tool can be handy for this comparison.
  * You compared the installed files of the new release to installed files of the previous stable release, and made sure that only expected modifications happened. You can compare these files by doing the following:
    1. Run the following command for the previous version:
```
./configure && make && make install DESTDIR=/tmp/ganetiA
```
    1. Run the following command for the current version:
```
./configure && make && make install DESTDIR=/tmp/ganetiB
```
    1. Compare both versions by running:
```
diff -rq /tmp/ganetiA /tmp/ganetiB | grep -v ^Files.*differ$
```

Note that you will get lots of messages like the following:

```
diff: /tmp/ganeti210/usr/local/share/man/man8/gnt-network.8: No such file or directory
diff: /tmp/ganeti211/usr/local/share/man/man8/gnt-network.8: No such file or directory
diff: /tmp/ganeti210/usr/local/share/man/man8/gnt-node.8: No such file or directory
diff: /tmp/ganeti211/usr/local/share/man/man8/gnt-node.8: No such file or directory
```

This is expected behavior since there are symlinks that currently point to nirvana due to the `$DESTDIR`.


# Prepare the release #
  1. If you haven't already, update `configure.ac` with the new version.
  1. Update the NEWS file with release date.
  1. Run `make distcheck` again.
  1. Set the environment variable for tag:
```
gnt_version=$(sed -n -e '/^PACKAGE_VERSION =/ s/^PACKAGE_VERSION = // p' Makefile); \
gnt_version_tag=$(echo $gnt_version | tr -d '~'); \
echo "Version '$gnt_version', tag '$gnt_version_tag'"
```
  1. Create the release tag:
```
git tag -m "Release version $gnt_version_tag" "v$gnt_version_tag"
git show "v$gnt_version_tag"
```
  1. Check the tag (with `--dry-run`), and then push it (removing `--dry-run`):
```
git push --tags --dry-run
```
  1. Create an archive from tag:
```
URL=git://git.ganeti.org/ganeti.git devel/release v$gnt_version_tag
```
  1. Upload the `.tar.gz` archive and (if needed) upload the documentation files to the project site: [downloads.ganeti.org](http://downloads.ganeti.org/). Releases must be signed, and the signature must be uploaded.
    * To sign (on the machine containing the signature key):
      1. `gpg --detach-sign <tar_gz_package>`
      1. `gpg --verify <signaturefile> <tar_gz_package>`
    * To create the checksum files:
      1. `md5sum <tar_gz_package> > <tar_gz_package>.md5sum`
      1. `sha1sum <tar_gz_package> > <tar_gz_package>.sha1sum`
  1. Remove the temporary dir used for creating the archive.
  1. Send the release announcement to [ganeti@googlegroups.com](mailto:ganeti@googlegroups.com).
  1. Update the topic on [irc://irc.freenode.net/ganeti](irc://irc.freenode.net/ganeti).
  1. ~~Update the [Freshmeat entry](http://freshmeat.net/projects/ganeti).~~
  1. Only for stable releases: Mark all fixed bugs with this branch as release target as released.

# Notes #

  * All changes must be reviewed and committed as usual.
  * Tagging cannot be reviewed, so have someone else review what you're doing.