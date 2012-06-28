gentoo-install
==============

An install script for Gentoo Linux

copyright
=========

Copyright (c) 2012, Michael Mol <mikemol@gmail.com>
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

Redistributions of source code must retain the above copyright notice, this
list of conditions and the following disclaimer.

Redistributions in binary form must reproduce the above copyright notice,
this list of conditions and the following disclaimer in the documentation
and/or other materials provided with the distribution.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
POSSIBILITY OF SUCH DAMAGE.

description
===========

In Gentoo installations, the installation sequence goes more or less this way:

1) Boot live environment
2) Create your system's filesystems
3) Unpack initial system binaries and package manager database
4) Configure your system's initial environment
5) chroot into your system
6) Configure your kernel and boot environment
7) Install (build) a few basic packages
8) Reboot
9) Finish building any remaining packages you need.
10) Add users
11) Use system

Even outside time spent compiling, this process is tedious (and thus error-
prone) and time-consuming. Further, I assume most Gentoo users are like myself
in that they spend a lot more time tweaking their already-configured systems
than installing new ones, so it's difficult to avoid missing steps during the install process.

This script seeks to contract the interactive portion of the installation sequence to:

1) Build your boot-time block devices and non-/ filesystems.
2) Provide a boot kernel and boot mechanism. (Build your /boot, kernel, and
   install grub)
3) Populate the script with configuration parameters, including a pre-formed
   make.conf file, and specifying any packages you would like initially
   available.
4) Run the script
5) Reboot
6) Add your users.
7) Use the system.

(Wouldn't it be beautiful if the script handled steps 1 and 2? But it doesn't.)

The script also tries to automate several cleanup behaviors:

* It tries to work around transient build failures such as those caused by race
  conditions in parallel make and package build orders by making extensive use
  of emerge's "--keep-going" and "--resume" features.
* It tries to get a head start on two-step rebuilds, where the first pass of
  building a set of packages and the second pass have different results,
  because of the versions of packages already present during the first pass.
  To get around this, it does a full "emerge -e @world" twice. This also has
  the effect of guaranteeing the entire system is using your specified CFLAGS,
  and also has been built with the latest marked-stable version of the
  compilers used.
* It runs dispatch-conf after every emerge, to ensure all configuration files
  are up to date.
* It runs revdep-rebuild after every emerge, to fix anything that may have
  broken. (Though this is unlikely, since we don't emerge --depclean. Still,
  the script is largely unattended, so the time spent isn't very important.)
* It runs perl-cleaner and python-updater after every emerge to guarantee
  bindings and modules in these languages are clean and up-to-date.

The script *may* fall into an infinite-rebuild-loop if an ebuild is broken. If
this happens, try ensuring your specified rsync server has an up-to-date copy
of portage. If it *still* happens, then file a bug! At some point, I'd like to
get this script to emit and save emerge --info data in the event of an emerge
failure, to help with the filing of these types of bug reports.

Oh, and the script spams your syslog with trace data. You can use
  tail -f /var/log/messages
in your live environment in order to keep track of what's going on.

sys-process/htop is also very fun.

caveats
=======

YOU WILL NEED TO MODIFY THE SCRIPT. It has several configuration variables
currently set in ways that are very specific to _my_ network.

This is all very rough at the moment, and it's strictly a "works for my
systems, on mt network" thing. Hopefully things improve and are generalized.

The script assumes the /boot and /home partitions already exist, are
formatted, and may be mounted by specifying the filesystems' UUID.
It also presumes that there's an HTTP proxy server available.

The script does not currently support preconfiguring arbitrary individual
packages (though this would be lovely, for distcc purposes), nor does it
currently support things like /etc/portage/packages.mask or
/etc/portage/packages.use . These would be wonderful enhancements.

The script ignores several parent environment variables, including http_proxy.
It would obviously be very beneficial to accept environment variables for
configuration. Perhaps in a future version.

Patches very welcome. The original git repository for this may be found at
github: https://github.com/mikemol/gentoo-install

