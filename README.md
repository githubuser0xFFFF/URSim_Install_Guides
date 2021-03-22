# URsim 5.10 Installation Guide for Ubuntu 16.04

The intention of this repository is to describe the installation of different
URSim versions on different Ubuntu versions. Currently this document
describes how to have **URSim 5.10** running in **Ubuntu 16.04.** The VM hat comes
from Universal Robots is an 32-bit Lubuntu 14.04.

If you have futher additions or an installation guide for a newer Ubuntu version,
feel free to create a pull request.

## Install Ubuntu 16.04

Download and install Ubuntu 16.04 from to [Ubuntu 16.04 releases page](https://releases.ubuntu.com/16.04/). The installation should work on the 32-bit and
64-bit version. 

To match the linux settings of the Lubuntu VM from Universal Robots, you should
use the following account settings:

- Hostname: **ursim**
- Username: **ur**
- Password: **easybot**

This guide was made with the settings above, so any changes there might cause
additional trouble.

## Download and unpack URSim 5.10

Download the URSim 5.10 packacge from the [Universal Robots Download Page](https://www.universal-robots.com/download). You need a user account for this download.

Extract the downloaded archive into the `/home/ur/ursim` folder. 

## Install runit package

The `install.sh` script will install the `runit` package - this will likely fail on Ubuntu 16.04. The install script already contains a note about this:

```bash
...
echo "Installing daemon manager package"
# if it fails comment out, and check answer https://askubuntu.com/a/665742
sudo apt-get -y install runit
...
```

So before we run the install script, we follow the instructions in the given
link to install runit: https://askubuntu.com/a/665742. Here are the steps
summarized:

Install runit: `sudo apt-get install runit`. If this succeeds, you can go to
the next step and skip the following fix. Most likely you will see the
folloing error:

```bash
Setting up runit (2.1.2-3ubuntu1) ...
start: Unable to connect to Upstart: Failed to connect to socket /com/ubuntu/upstart: Connection refused
dpkg: error processing package runit (--configure):
 subprocess installed post-installation script returned error exit status 1
Processing triggers for ureadahead (0.100.0-19) ...
Errors were encountered while processing:
 runit
E: Sub-process /usr/bin/dpkg returned an error code (1)
```

You should find a `runit.postinst` file in `/var/lib/dpkg/info/`. Edit this file
as root (sudo) in the following way:

Before:

```perl
if [ -x /sbin/start ]; then #provided by upstart
  /sbin/start runsvdir
fi
```

After:

```perl
#if [ -x /sbin/start ]; then #provided by upstart
#  /sbin/start runsvdir
#fi
```

When you've saved that change, you can tell apt to finish where it left off and you should be good to go:

```bash
sudo apt-get install -f
```

## Run URSim Install Script

Using your bash prompt run the `/home/ur/ursim/install.sh` using the following command.

```bash
sudo ./install.sh
```

If runit was installed properly before, the script should tun without any errors.

## Copy Libraries to /usr/bin

Since version 5.10 the extracted ursim folder contains the folder `usr/bin` with
executable files that should be installed to `/usr/bin` but which are not 
touched by the install script. Therfor we need to copy the files manually from
the usrim folder to `/usr/bin`.

CD into your ursim folder and copy the files:

```bash
sudo cp -avr ./usr/bin /usr
```

Now the command

```bash
ls /usr/bin/ur*
```

should output the following:

```bash
/usr/bin/uradmin
/usr/bin/uradmin-autorun
/usr/bin/uradmin-autorun-exec
/usr/bin/uradmin-control-mode
/usr/bin/uradmin-firewall-block-ports
/usr/bin/uradmin-firewall-restrict-incoming
/usr/bin/uradmin-firewall-service
/usr/bin/uradmin-help
/usr/bin/uradmin-passwd-check
/usr/bin/uradmin-passwd-set
/usr/bin/uradmin-print
/usr/bin/uradmin-ssh
/usr/bin/uradmin-ssh-add-authorized-key
/usr/bin/uradmin-ssh-remove-authorized-key
```

## Provide net-statistics Perl Script

For proper network support ursim requires the perl script `/sbin/net-statistics`.
I'm not a linux expert and I have no idea where this file comes from so I
simply copied this file from the URSim Lubuntu 14.04 VM to my Ubuntu 16.04
installation. After you have copied the file, you should make it executable:

```bash
sudo chmod u=rwx,g=rx,o=rx /sbin/net-statistics
```

Now you can test the script

```bash
perl /sbin/net-statistics
```

It should print out something like this:

```bash
Mode:static
Net up
Address:192.168.190.134
Mask:255.255.255.0
Gateway:
nameserver1:
nameserver2:
Hostname:ubuntu
```

If you do not see this, then you need to edit the file `/sbin/net-statistics` to adjust the name of the network interface. Use `ifconfig` to find out the name. This is how I changed the file on my system:

Before:

```perl
#!/usr/bin/perl

my $interface = "eth1";
my $isDhcp = 0;
my $isStatic = 0;
```

After:

```perl
#!/usr/bin/perl

my $interface = "ens33";
my $isDhcp = 0;
my $isStatic = 0;
```

## Edit /etc/network/interfaces

Check the mode value printed when executing the per script `/sbin/net-statistics`.

```bash
Mode:static
Net up
...
```

The mode value should be `static` or `dhcp`. If the value is `disabled` then you
need to edit your `/etc/network/interfaces` file, because the perl script
parses the values from this file.

```bash
sudo gedit /etc/network/interfaces
```

Because I use a static IP, I added the following lines:

```bash
# The primary network interface
auto ens33
iface ens33 inet static
address 192.168.190.134
netmask 255.255.255.0
```

After this change the perl script `/etc/network/interfaces` sucessfully printed
out the right mode `static`.

## Start URSim

Now you can start URSim from the command line:

```bash
./start-ursim.sh UR5
```

URSim should start without any errors now. If you see errors, then you can
check the log output on the console to get information what is going wrong.
