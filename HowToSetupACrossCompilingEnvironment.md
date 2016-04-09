# Introduction #

This document will discuss the necessary steps to get a cross-compiling environment built, it assumes you're an intermediate level Unix user. It may contain mistakes.


# Details #

If you're interested in building ARM binaries to run on your Archos and you've got a little Linux under your belt, you can do so pretty easily. While it may work on multiple distros, all of our examples will be written using CentOS, as it works and is what I'm using. If you like another distro you're welcome to give it a shot, any successes or failures you have can be added to this page, but by in large it should be pretty vanilla.

# Get Linux if you don't have it already #

As mentioned, I like CentOS, I'm using CentOS4.6 for this writing, you can download it from [one of these mirrors](http://isoredirect.centos.org/centos/4/isos/i386/). The default 'Developer' packages should be enough to get you the things you need. Setup your X and support packages however suits you best.

Don't waste your time with firewall rules or selinux, it'll just slow you down.

# Get the Archos GPL tarball #

You can download it from their website [here](http://www.archos.com/support/download/software/gpl_notice.html), get the tarball that fits your model. Since I'm an A605 user, I got the generation 5 tarball.

# Untar the GPL Tarball #

Anywhere will do, I put it in my home directory so that the subsequently created folder is available in ~/AX05\_GPL/

# Create your buildroot environment #

  1. cd ~/AX05\_GPL/buildroot
  1. cp configs/A605/defconfig .config
  1. make sconfig
  1. make

You will now have a built cramfs with the 'default' stuff that Archos puts on their builds (minus the non-gpl stuff), however, lets go into a bit more detail on what those steps are.

  * cd ~/AX05\_GPL/buildroot

> This step sets your current working directory to the 'buildroot' directory, this will be the nexus of most of your project builds.

  * cp configs/A605/defconfig .config

> This step copies the default Archos buildroot config into the buildroot directory. The .config file is similar to the Linux kernel config file, in it are the parameters with which your build is compiled. Like the linux kernel you can invoke a curses GUI to edit it by issuing the command 'make menuconfig' in the buildroot directory. To start, build the default just to make sure it works in your environment. Get fancy later.

  * make sconfig

> I don't know what this actually does, but it's some part of the build process. Probably like 'make deps' from Linux kernel. I'm too busy to dig into it.

  * make

> This will launch the whole process, build the tool-chain, build the arm binaries, and last of all create a cramfs (As configured by default) which will contain the root filesystem.

# I want to compile my own stuff! #

Well, start out by running 'make menuconfig', and browse the available packages. One item of note is that not all the packages are configured correctly, for instance berkeleydb uses an older version that isn't available on sleepycat.com, and, doesn't even goto sleepycat.com to get it (some mirror that also doesn't have the right version)

To fix that, you can go into ~/AX05\_GPL/package/

&lt;packagename&gt;

/ and edit it's .mk file, there you can modify the version and the host it tries to retrieve from, in most cases manually ftping and finding the next version tick up from what we wanted that wasn't available worked fine.

It should be noted that if a version isn't available, that likely means there was a security bug that got patched, otherwise it'd at least be available in an archival manner, so.. keep notes on what you need to replace, it may be your way back in after GFT gets patched.

# I know nothing about Linux, how do I make programs? #

You don't. Sorry.