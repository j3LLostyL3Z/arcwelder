# UPDATE #

  * GFT technique works on Archos 604, 605, and 705 wifi models

  * You can download a kitchen sink rootfs build (everything I could get to compile out of the Archos GPL tarball out of the box) from http://www.remix.net/archos/ax605_rootfs.tar.bz2

  * You can enable scp/sftp by either fixing the config to reference sftp/ssh from /tmp/ssh, or just hex edit the defaults in sshd. I haven't made this change in arcwelder-1.0.zip, this change will be made and a new archive will be uploaded (soon?).


# WARNING #

The techniques and software provided in this project are dangerous to the well-being of your Archos if you don't know what you're doing. If you're not comfortable in a Unix/Linux environment, DO NOT USE THIS SOFTWARE. You **will** brick your Archos and have nobody to blame but yourself.

In using these techniques and software you assume **all** risk and hold harmless the arcwelder project and myself for any damages real or imagined. Don't go crying to Archos either, it's not their fault.

To make things even more interesting, I've haven't tested things rigorously. Scared yet? You should be!

**_Nathan Ramella_, n ar @ h u sh.com**

# Summary #

Through a technique dubbed "Go Fighting Tabby!", or GFT for short, you can gain root access on an Archos 605 wifi running 1.7.13 firmware and execute arbitrary programs in its embedded Linux environment.

This is the first step to gain access to the running Linux operating system, which until now has not been possible.

Using the GFT technique, you can install the 'ARCwelder' package which will allow you to ssh into the Archos 605 wifi and run unix commands from a shell.

Currently ARCwelder only provides an ssh package, however in time more packages may be included as they are tested and verified as working, some customization is necessary to ensure that they work on the embedded system, so the term 'hack' is definitely applicable.

# Go Fighting Tabby! Technique Explained #

The GFT technique is pretty simple, after auditing the GPL tarball from Archos, I found a script named 'smbpasswdhelper'.

**smbpasswdhelper**
```
#!/bin/sh

if test $# -ne 2
then
        echo "usage: $0 account password"
        exit 1
fi

/opt/usr/bin/smbpasswd -x $1

/opt/usr/bin/smbpasswd -s -a $1 << EOF
$2
$2
EOF
```

What got me interested was that it didn't quote $2, I figured $1 would be hard-coded to whatever the smb user is for connecting (guest maybe?), so I started poking it a bit. As it turns out, that wasn't even necessary, as I believe the avos application is doing a straight system call without sanitizing the user supplied password string.

If you're at all familiar with shell commands, the prospect of an unsanitized string being used in a system() call is pretty attractive from a hacker's perspective. It means by using a little shell trickery you can launch your own command.

**But how do you use this script?**

That's the nice part, the Archos UI has provided you a text box to type in whatever you want for the 'password' under the wifi settings, after you've connected to a wifi network. You can set the password and launch an smb server, in doing so the smbpasswdhelper script is run (along with your hijack command). Even better, it runs your command as root, so you've got full privileges from the start.

# GFT Technique Applied #
  1. Connect to a wifi network
  1. From the top level of the avos app, goto your wireless settings again, this time you should see your IP address and other network configuration information. Scroll to the bottom and select **Wireless File Server..**
  1. Setup your Samba info as you like, but select the **Password** setting to bring up the keyboard UI.
  1. Change the password to **"1;ls -laR / >/mnt/data/Data/file\_list.txt"** , _include the quotes_, that's important
  1. Click/Run **Enable File Server**
  1. After a couple seconds, the UI will notify you that the fileserver has started, that means your command has run.
  1. Pop back up to the top level of the UI
  1. Select the **Files** icon by double clicking it
  1. Browse to **Data**, you should see file\_list.txt, you should be able to read the file by clicking it (If you have the Opera web browser, if you don't, you'll need to attach your Archos to a computer and read the file that way), inside file\_list.txt will be a full directory listing of your Archos device.
  1. You can run any supported Unix command using this technique, this gives you **direct access** to the "hidden" Linux partitions.
  1. This technique can be used to run / install other scripts, but if you install ARCwelder, you can just ssh in and do it that way.
  1. The Archos is set to power down the wireless whenever it can, so you may get disconnected if there's no activity. Since the archos support library is included in the GPL tarball, this may have something in it to hard-code longer timeouts.


# Installing ARCwelder #

  1. Download ARCwelder-1.0.zip
  1. Connect your Archos 605 wifi to your computer, select hard drive mode
  1. Mount your Archos on your computer
  1. Unzip the contents of ARCwelder-1.0.zip into 

&lt;ARCHOS&gt;

/Data/, this will make a directory named 'arcwelder' and populate it with the necessary files. When complete you can verify this step by checking 

&lt;ARCHOS&gt;

/Data/arcwelder/ and make sure that your file list matches the MANIFEST.txt file.
  1. Copy 

&lt;ARCHOS&gt;

/Data/arcwelder/id\_arcwelder into ~/.ssh/ on your client machine if you're using OSX or Linux, if you're using Windows download [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) and use that. id\_arcwelder is a password-less RSA key. For instructions on using PuTTY check out the wiki entry [How\_to\_use\_RSA\_keys\_with\_Putty](How_to_use_RSA_keys_with_Putty.md)
  1. DO NOT install id\_arcwelder.pub into your ~/.ssh/ directory.
  1. Use the GFT technique to run this command **"1;/bin/sh -x /mnt/data/Data/arcwelder/install"** , if you have trouble and need some debugging information you can GFT **"1;/bin/sh -x /mnt/data/Data/arcwelder/install 1>/mnt/data/Data/stdout.txt 2>/mnt/data/Data/stderr.txt"** , this will save both the standard output and standard errors into files so you can see what happened. A typical 'good' stderr file will look something like this:

```
+ mkdir /tmp/ssh/
+ mkdir /tmp/empty
+ chmod 755 /tmp/empty
+ cp /mnt/data/Data/ssh/authorized_keys /mnt/data/Data/ssh/fix /mnt/data/Data/ssh/id_archos /mnt/data/Data/ssh/id_arcwelder.pub /mnt/data/Data/ssh/sftp-server /mnt/data/Data/ssh/ssh-keysign /mnt/data/Data/ssh/ssh_banner /mnt/data/Data/ssh/ssh_config /mnt/data/Data/ssh/ssh_host_dsa_key /mnt/data/Data/ssh/ssh_host_dsa_key.pub /mnt/data/Data/ssh/ssh_host_key /mnt/data/Data/ssh/ssh_host_key.pub /mnt/data/Data/ssh/ssh_host_rsa_key /mnt/data/Data/ssh/ssh_host_rsa_key.pub /mnt/data/Data/ssh/sshd /mnt/data/Data/ssh/sshd_config /tmp/ssh/
+ cd /tmp/ssh
+ chmod 755 authorized_keys fix id_arcwelder id_arcwelder.pub sftp-server ssh-keysign ssh_banner ssh_config ssh_host_dsa_key ssh_host_dsa_key.pub ssh_host_key ssh_host_key.pub ssh_host_rsa_key ssh_host_rsa_key.pub sshd sshd_config
+ chmod 4711 ssh-keysign
+ chmod 600 /tmp/ssh/ssh_host_dsa_key /tmp/ssh/ssh_host_key /tmp/ssh/ssh_host_rsa_key
+ chmod 600 /tmp/ssh/authorized_keys
+ chmod 644 /tmp/ssh/id_arcwelder.pub /tmp/ssh/ssh_host_dsa_key.pub /tmp/ssh/ssh_host_key.pub /tmp/ssh/ssh_host_rsa_key.pub
+ /tmp/ssh/sshd -f /tmp/ssh/sshd_config
```

# Using ARCwelder #

  1. Connect to your wifi network using the instructions found on page 42 of the [Archos User Manual](http://www.archos.com/support/download/manuals/English-UserManual-ARCHOSGEN5-v2.pdf).
  1. Use the GFT technique to run the install script.
  1. Once ARCwelder is installed, and you have installed the private RSA key on your client machine, connect your Archos 605 wifi to your wireless connection, [Find\_your\_Archos\_IP\_address](Find_your_Archos_IP_address.md) and then run 'ssh -l root 

<archos\_ip>

', you should be greeted with the following banner and login prompt:

```
$ ssh -l root 192.168.1.120
                             _     _            
  __ _ _ __ _____      _____| | __| | ___ _ __ 
 / _` | '__/ __\ \ /\ / / _ \ |/ _` |/ _ \ '__| 
| (_| | | | (__ \ V  V /  __/ | (_| |  __/ |    
 \__,_|_|  \___| \_/\_/ \___|_|\__,_|\___|_|    
                                                


BusyBox v1.01 (2007.12.14-07:49+0000) Built-in shell (ash) 
Enter 'help' for a list of built-in commands. 

~ # 
~ # uname -a 
Linux (none) 2.6.10_mvl402 #2 Fri Dec 14 14:59:01 CET 2007 armv5tejl unknown 
```

# Caveats #

  1. If you've got any problems with this project, file an issue here -- don't email me, don't post in the forums, post it here.
  1. This project is not about bypassing the additional licensing fees for using codecs or web browser or whatever. This project provides NO tools to allow you to bypass these security measures and has no interest in doing so.
  1. If you brick your Archos, don't complain, you knew the risk, you took the risk. Backup your **entire** drive before you do anything if you're really concerned. If you don't know how to back up your entire drive, then you shouldn't use this software.
  1. Instead of offering me money for my efforts, contribute whatever you would have sent me to a local charity, if you actually do this write me a note to tell me what charity and how much and I'll add it to the list.

# Want to do more? #

If you're interested in taking this to the next level, you'll want to download the [Archos GPL](http://www.archos.com/support/download/software/gpl_notice.html?country=global&lang=en) tarball from their webpage. Once you have it you can create your own cross-compiling toolchain and create your own packages and potentially, even a distro if you're able to figure out how to make it boot without bricking your 605.