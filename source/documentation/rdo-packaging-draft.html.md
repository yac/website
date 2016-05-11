---
author: jruzicka
title: RDO OpenStack Packaging
---

# RDO Openstack Packaging

1. toc
{:toc}

## Introduction

This document attempts to be the definitive source of information about
*RDO* OpenStack packaging for developers and packagers.


### Packaging overview

**TODO**: `amoralej` is working on nice up-to-date diagrams with the new
workflow.


<a id="distgit"></a>

### distgit - where the .spec file lives

*distgit* is a git repository which contains `.spec` file used for building
a RPM package. It also contains other files needed for building source RPM
such as patches to apply, init scripts etc.

RDO packages' distgit repos are hosted on
[review.rdoproject.org](https://review.rdoproject.org/r/gitweb?p=openstack/nova-distgit.git;a=summary)
and follow `$PROJECT-distgit` naming.

You can use `rdopkg` to clone a RDO package `distgit` and also setup related
remotes:

```bash
$> rdopkg clone openstack-nova
Cloning distgit into ./openstack-nova/
git clone http://review.rdoproject.org/r/p/openstack/nova-distgit.git openstack-nova
...
```

Inspect package history using `git`:

```bash
$> cd openstack-nova
$> git checkout mitaka-rdo
$> git log --oneline
ded74f2 Add privsep-helper to nova sudoers file
55981cf Add python-microversion-parse dependency
39d576a Update .gitreview
4e53ad0 Add missing python-cryptography BuildRequires
```

See what `rdopkg` thinks about current distgit:

```bash
$> rdopkg pkgenv

Package:   openstack-nova
Version:   13.0.0
Upstream:  13.0.0
Tag style: X.Y.Z

Patches style:          review
Dist-git branch:        mitaka-rdo
Local patches branch:   mitaka-patches
Remote patches branch:  patches/mitaka-patches
Remote upstream branch: upstream/master
Patches chain:          http://review.rdoproject.org/r/631
```

Submit distgit changes for review:

```bash
$> rdopkg review-spec
```

#### CloudSIG vs RDO Trunk

**TODO**: explain `rpm-{master,$RELEASE}` vs `$RELEASE-rdo`. Not sure where to
put this in the doc...


<a id="patches-branch"></a>

### Patches branch

Because we rebase and backport often, manual management of patch files in
distgit would be _unbearable_. That's why each distgit branch
has an associated **patches branch** which contains upstream git tree with
extra downstream patches on top. 

A distgit can be automatically updated by `rdopkg` to include patches from
associated **patches branch** and thus RPM patches are managed with `git`. 

Individual RDO patches are maintained in form of gerrit reviews on
[review.rdoproject.org](https://review.rdoproject.org/r/#/q/status%3Aopen+project%3Aopenstack/nova).

You can check out package's patches branch using `rdopkg get-patches` if you
have remotes set up correctly (`rdopkg clone` sets them up for you):

```bash
$> git checkout mitaka-rdo
$> rdopkg get-patches
```

Send new/modified patches for review using `rdopkg review-patch`:

```bash
$> rdopkg review-patch
```

#### patches branch workflow

```
     +------------------------+
     |        upstream        |
     |  github.com/openstack  |
     +------------------------+
                 |
 git cherry-pick | rdopkg review-patch
                 V
     +-------------------------+
     |      patches branch     |
     |  review.rdoproject.org  |
     +-------------------------+
                 |
    rdopkg patch | rdopkg review-spec
                 V
     +-------------------------+
     |        distgit         |
     |  review.rdoproject.org  |
     +-------------------------+
```


<a id="rdopkg"></a>

### rdopkg

[rdopkg](https://github.com/redhat-openstack/rdopkg) is a command line tool
that automates many operations on RDO packages including:

 * cloning package distgit and setting up remotes
 * introducing patches
 * rebases to new versions
 * sending changes for review
 * querying `rdoinfo` metadata
 * modifying .spec file: bumping versions, managing patches, writing
   changelog, producing meaningful commit messages, ...

`rdopkg` is a Swiss army knife of RDO packaging and it automates a number of
repetitive and error prone processes involving several underlying tools, each
with its own quirks.

Install `rdopkg` from [`jruzicka/rdopkg` copr](https://copr.fedorainfracloud.org/coprs/jruzicka/rdopkg/):

```bash
$> dnf copr enable jruzicka/rdopkg 
$> dnf install rdopkg
```

`rdopkg` source lives at
[review.rdoproject.org](https://review.rdoproject.org/r/gitweb?p=rdopkg.git;a=summary)
but it's also mirrored to
[github](https://github.com/redhat-openstack/rdopkg).

Bugs are tracked as
[github issues](https://github.com/redhat-openstack/rdopkg/issues).

Poke `jruzicka` on `#rdo` for help/hate/suggestions about `rdopkg`.

See also [man rdopkg](https://www.rdoproject.org/packaging/rdopkg/rdopkg.1.html).


<a id="rdoinfo"></a>

### rdoinfo metadata

`rdoinfo` is a git repository containing RDO packaging metadata such as
releases, packages, maintainers and more, currently in a single `rdo.yml` YAML
file.

`rdoinfo` lives at
[review.rdoproject.org](https://review.rdoproject.org/r/gitweb?p=rdoinfo.git;a=summary)
but it's also mirrored to
[github](https://github.com/redhat-openstack/rdoinfo).

To query `rdoinfo`, use `rdopkg info`:

```bash
$> rdopkg info
$> rdopkg info openstack-nova
$> rdopkg info maintainers:jruzicka@redhat.com
```

To integrate `rdoinfo` in your software, use `rdopkg.actionmods.rdoinfo`
module.


### DLRN

**TODO**


## RDO Trunk Packaging Guide

RDO Trunk repositories provide packages of latest upstream code without any
additional patches. Packages are built automatically by [DLRN](#DLRN) from
`.spec` templates residing in `rpm-master` and `rpm-$RELEASE` [distgits](#distgit).

**TODO**


## RDO CloudSIG Packaging Guide

RDO CloudSIG OpenStack repositories provide packages of upstream stable
releases. This is kind of "stable RDO" living in `$RELEASE-rdo`
[distgit](#distgit). Patches can be introduced as needed through associated
[patches branch](#patches-branch).

**TODO**: links to repos?


### Prerequisites

 * You need a [github](https://github.com) account which is used to log into
[review.rdoproject.org](http://review.rdoproject.org).
 * You need to install [rdopkg](#rdopkg) tool.


### Initial repository setup

`rdopkg clone` takes care of getting the package [distgit](#distgit) and
also setting up all relevant git remotes defined in [rdoinfo](#rdoinfo).
Use `-u`/`--review-user` option to specify your github username if it differs
from `$USER`.

```bash
$> rdopkg clone openstack-nova -u github-username
Cloning distgit into ./openstack-nova/
git clone http://review.rdoproject.org/r/p/openstack/nova-distgit.git openstack-nova
Adding patches remote...
git remote add patches http://review.rdoproject.org/r/p/openstack/nova.git
Adding upstream remote...
git remote add upstream git://git.openstack.org/openstack/nova
...
```

Check output of `rdopkg pkgenv` to see what `rdopkg` thinks about your
package:

```bash
$> cd openstack-nova
$> git checkout mitaka-rdo
$> rdopkg pkgenv
```


### Simple `.spec` fix

The simplest kind of change that doesn't introduce/remove patches or
different source tarball.

* Make required changes.
* Bump `Release`.
* Provide useful `%changelog` entry describing your change.
* Commit the [distgit](#distgit) changes with meaningful commit message.
* Send the change for review.

Although this change is simple, `rdopkg fix` can still make some string
manipulation for you. In following example, I add a new dependency to `nova`
package:

```bash
$> cd openstack-nova
$> git checkout mitaka-rdo
$> rdopkg fix

Action required: Edit .spec file as needed and describe changes in changelog.

Once done, run `rdopkg -c` to continue.

$> vim openstack-nova.spec
# Add Requires line and describe the change in %changelog
$> rdopkg -c
```

After this, `rdopkg` generates new commit from the %changelog entry you
provided and displays the diff:

```diff
    Epoch:            1
    Version:          13.0.0
   -Release:          1%{?dist}
   +Release:          2%{?dist}
    Summary:          OpenStack Compute (nova)
    
    ...

    Requires:         bridge-utils
    Requires:         sg3_utils
    Requires:         sysfsutils
   +Requires:         banana
    
    %description compute
    OpenStack Compute (codename Nova) is open source software designed to

    ...
    
    %changelog
   +* Mon May 09 2016 Jakub Ruzicka <jruzicka@redhat.com> 1:13.0.0-2
   +- Require banana package for the lulz
   +
    * Thu Apr  7 2016 Haïkel Guémar <hguemar@fedoraproject.org> - 1:13.0.0-1
    - Upstream 13.0.0
```

Finally, send the changes for review:

```bash
$> rdopkg review-spec
```


### Introducing/removing patches

See [patches branch](#patches-branch) for introduction.

First, use `rdopkg get-patches` to get a [patches branch](#patches-branch)
associated with current [distgit](#distgit), cherry pick your patch(es) on
top, and send them for review with `rdopkg review-patch`:

```bash
$> git checkout mitaka-rdo
$> rdopkg get-patches
$> git cherry-pick YOUR_PATCH
$> rdopkg review-patch
```

Once the patch gets approved (not merged), you can tell `rdopkg` to update the
distgit and send the `.spec` change for review:

```bash
$> git checkout mitaka-rdo
$> rdopkg patch
$> rdopkg review-spec
```


### Rebasing on new version

tl;dr `rdopkg new-verison` should take care of that:

```bash
$> git checkout mitaka-rdo
$> rdopkg new-version
```
