# URsim 5.10 Installation Guide for Lubuntu 20.04

## Install Lubuntu 20.04

Download and install Lubuntu 20.04 from [Lubuntu releases page](https://cdimage.ubuntu.com/lubuntu/releases/).

## Download URSim 5.12.2 Debian package

You can download a 5.12.2 debian package from the following GitLab repository:

https://gitlab.com/gitlabuser0xFFFF/ursim-debian-packages

Browse the [CI/CD pipelines](https://gitlab.com/gitlabuser0xFFFF/ursim-debian-packages/-/pipelines?page=1&scope=branches#) and just download the [build artifacts](https://gitlab.com/gitlabuser0xFFFF/ursim-debian-packages/-/jobs/4153850155/artifacts/download?file_type=archive) for the **ubuntu-2004-ursim-5.12.2** branch.

```bash
wget https://gitlab.com/gitlabuser0xFFFF/ursim-debian-packages/-/jobs/4153850155/artifacts/download?file_type=archive
```

## Install the Debian package

Unpack the build artifacts archive and install the debian package:

```bash
sudo apt install ./*.deb
```

## Provide net-statistics Perl Script

For proper network support ursim requires the perl script `/sbin/net-statistics`.
This seems to be a custom script generated for the URSim, but not provided in the installation. The script is located in the src directory of this repository.
Download the file, move it to the correct location and add executable permisions:

```bash
wget https://github.com/githubuser0xFFFF/URSim_Install_Guides/raw/ubuntu-2004-ursim-5.12.2/src/net-statistics
sudo mv net-statistics /sbin/
sudo chmod +x /sbin/net-statistics
```

Now you can test the script

```bash
net-statistics
```

It should print out something like this:

```bash
Mode:dhcp
Net up
Address:192.168.190.134
Mask:255.255.255.0
Gateway:
nameserver1:
nameserver2:
Hostname:ursim
```

If you do not see this, then you need to edit the file `/sbin/net-statistics` to adjust the name of the network interface. Use `ifconfig` to find out the name. This is how I changed the file on my system:

```bash
sudo vim /sbin/net-statistics
```

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

my $interface = "enp0s3";
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
sudo vim /etc/network/interfaces
```

Because I use a static IP, I added the following lines:

```bash
# The primary network interface
auto enp0s3
iface enp0s3 inet static
address 10.0.2.15
netmask 255.255.255.0
```

After this change the perl script `/etc/network/interfaces` sucessfully printed
out the right mode `static`.

If you prefer DHCP you can configure the interface like this:

```bash
# The primary network interface
auto enp0s3
iface enp0s3 inet dhcp
```

## Start URSim

Now you can start URSim from the command line. The debian package installed
ursim into `/opt/ursim` folder. So you can start it via:

```bash
/opt/ursim/5.12.2/start-ursim.sh UR5
```

URSim should start without any errors now. If you see errors, then you can
check the log output on the console to get information what is going wrong.

## Create desktop icon

You can add a desktop icon to start URsim from desktop:

```ini
[Desktop Entry]
Version=5.12.2.1101534
Type=Application
Terminal=false
Name=ursim-5.12.2.1101534 UR5
Exec=/opt/ursim/5.12.2/start-ursim.sh UR5
Icon=/opt/ursim/5.12.2/ursim-icon.png

```

## Add to autostart

If you would like to autostart URsim in your VM, just copy the just created
icon file from the desktop to the `.config/autostart` folder in your home
directory.

## Configure Firewall for remote control

By default, Ubuntu has a built-in firewall: UFW, which stands for "Uncomplicated Firewall".
To access the UR robot ports remotely, you need to properly configure this
firewall. Here is the list of [UR client interfaces with port numbers](https://www.universal-robots.com/articles/ur/interface-communication/overview-of-client-interfaces/).

The quick and dirty solution is, to simply allow all incoming connections:

```bash
sudo ufw default allow incoming
sudo ufw default allow outgoing
```

If you would like to have better security, you need deny all incoming connections
and then properly allow all [ports]((https://www.universal-robots.com/articles/ur/interface-communication/overview-of-client-interfaces/)) required by UR robot:

```bash
sudo ufw default deny incoming
sudo ufw allow ssh
sudo ufw allow 29999
...
```

If you prefer a UI for firewall configuration, you can install a graphical
frontend:

```bash
sudo apt install gufw
```
