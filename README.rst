build
=====

A buildroot is a root filesystem designed to be used as a chroot to build
packages within.

It assumes that you have already configured a bootstrap. This should already
exist on TKLDev by default. Otherwise please see the `bootstrap`_ repo.

Build buildroot for current release
-----------------------------------

This requires that the TurnKey dependencies have been built and uploaded to the
TurnKey repos (this should generally be the case).::

    make clean
    make

By default ``make`` builds to the ``pkg_install`` target, if you intend on using
this buildroot directly, you can build straight to the ``install`` target (which
installs the buildroot to ``$FAB_PATH/buildroots/$(basename $RELEASE)-$FAB_ARCH``::

    make install

Build buildroot for transition (new release)
--------------------------------------------

When doing a distro transition (i.e. preparing for a new major version release
- e.g. moving from one Debian release to the next), things are a little less
straight forward. There are a number of TurnKey specific packages that are
required and may not yet be built. If it's very early in the release, then the
relevant TurnKey repos may not even exist yet.

Before the relevant TurnKey apt repos exist, all TurnKey apt repos can be
disabled by setting NO_TURNKEY_APT_REPO=y (all TKL apt lines will be commented
out). E.g.::

    export RELEASE=debian/::CODENAME::
    make clean
    NO_TURNKEY_APT_REPO=y make install

Note that this will build each of the TurnKey dependencies from source. If the
source code isn't already available locally (in '/turnkey/public/${pkg}') it
will be cloned there from GitHub (assuming internet access).

If all the required TurnKey dependencies are available, but only in the
turnkey-testing repo (as is likely early in the transition process), then
instead set TKL_TESTING=y. E.g.::

    export RELEASE=debian/::CODENAME::
    make clean
    TKL_TESTING=y make install

If the TurnKey apt repos exist and the relevant packages are in the main
TurnKey apt repo, then beyond the need to set the RELEASE, building for a
transition is essentially the same as a normal build. I.e.::

    export RELEASE=debian/::CODENAME::
    make clean
    make install

Building on a vanilla Debian system
-----------------------------------

It is possible to build on a vanilla Debian system, however currently root is
needed, so run with sudo. This also means that any build-env vars need to be
explicitly passed via the commandline. E.g.:

    sudo RELEASE=debian/trixie FAB_PATH=$FAB_PATH make clean
    sudo RELEASE=debian/trixie FAB_PATH=$FAB_PATH OTHER_ENV_VARS=... make install

Ideally things will be updated to run as a normal user at some point, but until
then, please be aware that the resulting buildroot will be owned by root.

See below for other available env vars.

Copy generated buildroot to buildroots folder
---------------------------------------------

Once the buildroot is built, then it needs to be copied to the desired
location (default: ${FAB_PATH}/buildroots/::CODENAME::).

As noted above, whether building a transition or not, the ``install`` target
does this for you. I.e.::

    make install

Build env vars
--------------

RELEASE=                - release to build; DISTRO/CODENAME
                          e.g. 'debian/trixie'
BOOTSTRAP=              - path to bootstrap dir
                          (default: $FAB_PATH/bootstraps/CODENAME-$FAB_ARCH)
NO_TURNKEY_APT_REPO=y   - disable TurnKey apt repositories completely
TKL_TESTING=y           - enable $RELEASE-testing apt repository
APT_PROXY_OVERRIDE=     - set custom proxy port (or 'disable' to disable proxy)
NO_PROXY=y              - same as APT_PROXY_OVERRIDE='disable'
FAB_ARCH=               - architecture to build; defaults to host arch
                          current supported arch: amd64, arm64
FAB_SHARE_PATH=         - path to fab share directory (where product.mk can be
                          found - defaults to /usr/share/fab)

.. _bootstrap: https://github.com/turnkeylinux/bootstrap
