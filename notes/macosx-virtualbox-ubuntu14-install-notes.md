# Notes on installing Ubuntu guest VMs in VirtualBox on a macOS host

2017-Apr-07: On Mac OS X 10.11.6 system,
installed VirtualBox 5.1.18 r114002,
and created new Ubuntu 14.04.1 64-bit VM

Also an update from 2017-Nov-07:
On macOS Sierra 10.12.6 system,
installed VirtualBox 5.1.30 r118389,
and created Ubuntu 16.04.3 64-bit desktop VM

Removed several icons from Launcher that I didn't want
(e.g. LibreOffice), by right-clicking on them and choosing the popup
menu item to unlock them from the launcher.

Click on the top icon on the Launcher, which creates a text box.
Enter "terminal" to search for that program.  Click "Terminal" icon to
start that program.  Right-click on Terminal icon in Launcher and
select "Lock to Launcher" so it will always be there.

```bash
# Do this only if you want Andy Fingerhut's personalized dot
# files.  You probably want your own instead.  WARNING: Do not
# skip the export command.  Shell scripts executed in the step
# after that need it to be set, or else by default they will get
# files from a different Github repo belonging to someone else.

% sudo apt-get install curl
% export github_user=jafingerhut
% bash -c "$(curl -fsSL https://raw.github.com/$github_user/dotfiles/master/bin/dotfiles)" && source ~/.bashrc

# (optional) Save some disk space by removing large programs I won't use
% sudo apt-get purge thunderbird libreoffice-*

# Useful packages I like to have, not installed by default.
% sudo apt-get install python-pip python3-pip
```


# Getting copy/paste working between host and guest OS

The older approach I used for Ubuntu 14.04 is described later below,
involving installing virtualbox-guest-dkms and virtualbox-guest-x11
packages.  Search for "Approach installing virtualbox-guest-*
packages"


## Approach installing Guest Additions via VirtualBox CD image

[Thanks to Sean Adams for these working instructions.]

Start VirtualBox and boot the Ubuntu VM without a CD/DVD in the
"drive".  Log in to the desktop (assuming you're using the desktop
edition).  Now on the Mac go to the Devices menu of VirtualBox.  The
last item will be "Install Guest Additions CD image...".  Select it.
In the VM, a CD will appear in the dock and a dialog should pop up
asking if you want to allow the software on the CD to autorun.  Let it
run.  A second dialog will ask you to authenticate (sudo) to install
the software.  Once it is done, shutdown and reboot the VM.  When the
VM comes back up and the bidirectional clipboard is enabled, copy and
paste should work as expected.

The Guest additions change with every version of VirtualBox.  Be
prepared to re-install them whenever you upgrade VirtualBox.

Even minor updates trigger a message in the VM suggesting to update.
I'm not sure it is required, but I do it anyway.

Following the instructions above re-installs the guest additions even
if they have already been installed in the VM.


## Approach installing virtualbox-guest-* packages

I was able to get host/guest copy and paste working using this
approach for Ubuntu 14.04.1 guest OS.  See above for steps for an
Ubuntu 16.04 guest OS.

    # This might be necessary to get copy/paste to work between guest
    # and host OS
    sudo apt-get install virtualbox-guest-dkms

Reboot system for newly installed kernel modules (installed by
virtualbox-guest-dkms package) to take effect.  Larger display size
choices are now available in System Settings -> Displays

To get other custom display sizes not on that menu, just click and
drag the bottom right corner of the VirtualBox window to resize it,
then wait several seconds until the guest Ubuntu desktop changes to
match it.

Way more details than one probably would like on this issue:

    http://askubuntu.com/questions/451805/screen-resolution-problem-with-ubuntu-14-04-and-virtualbox/595192#595192


With Ubuntu 14.04 I could get copy and paste between guest and Mac OS
X to work.  With Ubuntu 16.04.2 I could not.  It was failing to
install package virtualbox-guest-x11.  Some more details in this
askubuntu.com question, but I haven't tried the suggestion of removing
the package libcheese-gtk23 yet, because when I try to do that via
synaptic, it warns that many other packages need to be removed, also,
and some of them like ubuntu-desktop look like things I want to keep.

    https://askubuntu.com/questions/526995/installing-guest-additions-causing-problems

Shut down guest Ubuntu OS.


# Solving screen flicker issue with Ubuntu 17.10 guest VM in VirtualBox

Found this solution here: https://forums.virtualbox.org/viewtopic.php?f=8&t=85110

This is the comment that I found helped:

    I found this to be a bug with the new Ubuntu 17.10 in Sierra and
    VBox 5.2.0 and fixed it by selecting Xorg instead of default
    Wheland window system. You will find this option in the Ubuntu
    login dialog under the gear.



# Set up shared folders between Ubuntu guest OS and host macOS

I was able to get shared folders working for both Ubuntu 14.04 and
Ubuntu 16.04 guest OS's using these instructions.

I believe I had trouble with shared folder settings disappearing if I
tried creating them while the guest OS was running.  Shutting down the
guest OS and then creating the shared folder settings seemed to work
better, so do that first.

Menu item: Machine -> Settings... -> Shared Folders tab
Click icon on right side of window with + sign in it
Select the desired folder on host OS
Click to enable "Auto-mount"
Click OK to save settings.

The folder name I created was 'p4-docs', which I will use below for
sake of example.

Boot guest OS and log in.

```bash
% ls /media
```

should show a new directory sf_p4-docs/
Trying to list it will likely give 'Permission denied'.

Command to add a user to a group:

```bash
% sudo usermod -a -G <groupName> <userName>
```

In this particular case for the group 'vboxsf':

```bash
% sudo usermod -a -G vboxsf $USER
```

Log out and log back in, or shut down and reboot guest OS.  Run the
command 'id' to confirm that the group 'vboxsf' now shows up in the
output.  If so, then this command should now show you the contents of
the shared folder:

```bash
% ls /media/sf_p4-docs
```

A symbolic link to this shared folder can make it easy to access from
your home directory, e.g.:

```bash
% ln -s /media/sf_p4-docs/ ~/p4-docs
```


# Shrinking a guest OS disk image

A StackOverflow Q&A on compacting/shrinking a guest OS VDI file
system:

    https://superuser.com/questions/529149/how-to-compact-virtualboxs-vdi-file-size


# Sample sshfs commands

If all you want is to share a directory between host and guest OS, I
would recommend the previous section to set that up.  Trying to use
sshfs for this with a host that is a laptop with a changing IP address
(e.g. by connecting to a different Wi-Fi access point) causes the
sshfs connection to break and require re-creating, which I find
annoying.

sshfs is nice in that you do not need to have root privileges on any
machines to use it -- only the ability to ssh from the machine
mounting the remote file system, to the machine where the file system
is located.

It is not so convenient when used to mount a host OS's file system on
a guest OS.

(a) One way is to use the IP address of the host OS's main network
    adapter.  This works well, except that this IP address changes if
    the host OS is on a laptop, and its Wi-Fi interface gets a new IP
    address after changing locations.

(b) Another way I have successfully used is to create a host-only
    network (instructions below).  This prevents the IP address from
    changing, but unfortunately does not work when the host OS is
    currently using Cisco AnyConnect with an active VPN connection,
    because that software prevents 'split tunneling', meaning that no
    IP traffic on the host can go to any network interface other than
    the virtual VPN interface, so no IP communication between guest
    and host during those times.

It is fine for mounting a file system of a machine with a consistent
IP address on the host or the guest.

```bash
# mounting
# sshfs [user@]host:[dir] mountpoint [options]

% MYIP="10.0.1.40"
% MYIP="192.168.56.1"
% sshfs jafinger@jafinger-aur.cisco.com:/auto/ins-asic-11/jafinger/forwarding/ins-asic $HOME/ins-asic

# unmounting
# fusermount -u <mountpoint>
% fusermount -u $HOME/ins-asic
```


# Create host-only adapter network between guest and host OS

Create host-only adapter network between guest Ubuntu VM and host Mac
OS X, so that host OS will have consistent IPv4 address that guest
Ubuntu VM can use to sshfs mount directories physically stored in the
host Mac OS X file system, with a consistent IPv4 address that does
not depend on where the host Mac OS X laptop is, and thus which IPv4
address its Wi-Fi adapter happens to have.

In VirtualBox, create a Host-only adapter if you haven't done so
before.  I followed instructions here:

    https://www.virtualbox.org/manual/ch06.html#network_hostonly

* In Mac OS X VirtualBox application, choose menu item VirtualBox ->
  Preferences...

* Click Network
* Click Host-only Networks
* Click icon on right that has hover-text help "Adds new host-only network"
* I used default name "vboxnet0"
* In menu, right click vboxnet0 and choose "Edit Host-only Network"
* In window that appears, I saw under Adapter tab:

IPv4 Address: 192.168.56.1
IPv4 Network Mask: 255.255.255.0
IPv6 Address: <blank>
IPv6 Network Mask Length 0 (grayed out)

I left everything as shown above.  Under "DHCP Server" tab:

    Check box next to "Enable Server" was checked.
    Server Address: 192.168.56.100
    Server Mask: 255.255.255.0
    Lower Address Bound: 192.168.56.101
    Upper Address Bound: 192.168.56.254

click Settings for the VM.

System -> Processor -> change to 4 virtual CPUs.  1 is the default.

    Network -> Adapter 2
    enable check box next to "Enable Network Adapter"
    in popup menu next to "Attached to:" choose "Host-only Adapter"
    In popup menu next to "Name:" choose vboxnet0

Click OK.  Settings should be saved.

In host Mac OS X system, 'ifconfig' in a terminal should show an
interface "vboxnet0" with same IPv4 address as shown above
(192.168.56.1).

Boot up guest Ubuntu VM.

In guest OS, 'ifconfig' in a terminal should show eth1, the 2nd
network adapter (first is called eth0) with an IPv4 address somewhere
in the range 192.168.56.101 through 192.168.56.254 (probably the first
in that range).

From guest OS, should be able to ping host OS at 192.168.56.1.  Also
should be able to ssh to it, if host Mac OS X is enabled for "Remote
Login" under System Preferences -> Sharing

From host OS, should be able to ping guest OS at 192.168.56.101.  I
don't have my guest Ubuntu VM configured to allow ssh into it.

Note: Unfortunately, if the host is Mac OS X, and you run Cisco
AnyConnect Secure Mobility Client on that host to connect via VPN,
while you are connected via VPN, the host only network adapter will
not be able to send or receive packets between the host and the guest.
Bummer.  Shared folders seems like a better way to go, then, for
sharing files on the host file system with the guest OS.


# Notes on VirtualBox and VMware Fusion on OSX 10.6.8 Snow Leopard

The only reason for this section is that I have an older MacPro model
MacPro2,1 that cannot run any more recent versions of OSX than 10.6.8
(maybe 10.7.x, maybe not).  It isn't the fastest machine as of 2017,
but it does have 32 GB of RAM, which is still the most RAM in one
machine I own.  I can run Linux on it as the host OS, but I am curious
to know about running recent versions of Linux as a guest VM.

I have a copy of VMware Fusion 3.1.4 that I bought years ago, and
installed there.  I found recently that when upgrading Ubuntu 16.04 to
the most recent version as of Oct 2017, the GUI no longer worked.
Only console-based access was available in the VM, which is annoying.

Error message I see during boot of guest before GUI fails:

    nsc-ircc, Wrong chip version 00


I believe that VirtualBox 4.3.40 is the latest version supported on
OSX 10.6.8 hosts.  I have a copy of that installed, and I can run the
most recent 2017-Oct versions of Ubuntu 16.04 in it, but with at most
2 GB of guest OS RAM.  This is usable, but I'd like to do better.  I
have tried it with 2.5 GB and 3 GB of guest OS RAM, and the Ubuntu
install steps failed with an error from VirtualBox about not enough
memory being available on the host OS.  There is certainly enough
physical RAM, so this might be some kind of limitation with virtual
memory in that version of VirtualBox.  Not sure why that happens.

I did a little bit of Google searching for the error message, and some
people experienced problems on 64-bit Windows host OS's with guest
Linux VM, when the host had a particular range of versions of the
Google Chrome browser, which installed some 64-bit application that
may have been preventing VirtualBox from allocating as much physically
contiguous RAM as it wanted.  That is just a guess from reading the
descriptions of the problem briefly.



I found a VMware Fusion page describing which versions of VMware
Fusion are supported on which versions of Mac OSX, and found that
VMware Fusion 5 is the latest one supported, and perhaps only for OSX
10.6.8 (i.e., no earlier versions of OSX 10.6.x supported).

I will soon try installing that version of VMware Fusion to see if it
allows me to run the latest versions of Ubuntu 16.04, with GUI, and
relatively large guest OS memory (e.g. 8 GB or maybe even more).


Of course I can also just run Ubuntu as the host OS, too.
