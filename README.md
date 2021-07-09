# URsim 5.10 Installation Guide for Ubuntu 18.04

Currently this document describes how to install **URSim 5.10** in **Ubuntu 18.04.** 

If you have futher additions, feel free to create a pull request.

## Install Ubuntu 18.04

Download and install Ubuntu 18.04 from to [Ubuntu 18.04 releases page](https://releases.ubuntu.com/18.04/).


## Download and unpack URSim 5.10

Download the URSim 5.10 packacge from the [Universal Robots Download Page](https://www.universal-robots.com/download/software-e-series/simulator-linux/offline-simulator-e-series-ur-sim-for-linux-5100/). You need a user account for this download.

Extract the downloaded archive as per the instructions.

# Dependency requirements

The scipt needs Java 6-8 installed, but automatically installs Java 11. If Java 8 is installed prior to running the install script it won't install Java 11. Do so by running: `sudo apt install openjdk-8-jdk openjdk-8-jre`

## Run URSim Install Script

Using your bash prompt run the `install.sh` script using the following command.

**Note: If you have ROS installed, running the following script will wipe your install.**

```bash
sudo ./install.sh
```

## Add binaries to PATH

```bash
echo -e "\nPATH="$PATH:$HOME/ursim-5.10.0.106288/usr/bin" >> ~/.profile
source ~/.profile
```


## Provide net-statistics Perl Script

For proper network support ursim requires the perl script `/sbin/net-statistics`.
This seems to be a custom script generated for the URSim, but not provided in the installation. The script is located in the src directory of this repository.
Download the file, move it to the correct location and add executable permisions:

```bash
wget https://raw.githubusercontent.com/ljden/URSim_Install_Guides/main/src/net-statistics
sudo mv net-statistics /sbin/
sudo chmod +x /sbin/net-statistics
```

Now you can test the script

```bash
net-statistics
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

Now you can start URSim from the command line:

```bash
./start-ursim.sh UR5
```

URSim should start without any errors now. If you see errors, then you can
check the log output on the console to get information what is going wrong.

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
