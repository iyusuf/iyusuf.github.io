## Install Utilities
### bashit
- Install:
- ref: https://github.com/Bash-it/bash-it
- Check out a clone of this repo to a location of your choice, such as: git clone --depth=1
https://github.com/Bash-it/bash-it.git ~/.bash_it
- Run ~/.bash_it/install.sh (it automatically backs up your ~/.bash_profile or ~/.bashrc, depending on your
OS)
- source .bashrc
- Theme:
- ref : https://github.com/Bash-it/bash-it/wiki/Themes
-

### wget
- sudo yum install wget
gradle
- ref https://gist.github.com/parzonka/9371885
- it worked after a reboot
- Dont know how though.


### maven
- look at useful sheel script or the new name i think is “linux-scriptsjdk-

#### Install
```
$ wget --no-cookies --no-check-certificate --header "Cookie:
gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie"
"http://download.oracle.com/otn-pub/java/jdk/8u112-b15/jdk-8u112-linux-x64.rpm"

sudo yum localinstall jdk-8u112-linux-x64.rpm -y

rm jdk-8u112-linux-x64.rpm
```

#### Uninstall

```
$ sudo yum remove jdk
```

## Networking

### hostname

### Activate Connection
- [Ref:](http://www.krizna.com/centos/setup-network-centos-7/)
- Run nmtui
- Activate
- Check Automatic
‘ifconfig’
- [Ref:](https://www.unixmen.com/ifconfig-command-found-centos-7-minimal-installation-quick-tip-fix/)
- By default ‘ifconfig’ is not installed.
- You can use ‘ip addr’
- Find the local guest ip address and it should show 10.0.2.15 for VirtualBox ‘NAT’ connection.

### static ip
- Create a file named /etc/sysconfig/network-scripts/ifcfg-enp0s3 as follows:

```bash

	BOOTPROTO=static
	DEFROUTE=yes
	IPV4_FAILURE_FATAL=yes
	IPV6INIT=yes
	IPV6_AUTOCONF=yes
	IPV6_DEFROUTE=no
	IPV6_FAILURE_FATAL=no
	IPV6_ADDR_GEN_MODE=stable-privacy
	NAME=enp0s3
	#UUID=c27cf49c-2cde-4286-a831-49f11daf9e79
	DEVICE=enp0s3
	ONBOOT=yes
	PREFIX=24
	IPADDR=10.0.2.51
	PEERDNS=yes
	PEERROUTES=yes
	IPV6_PEERDNS=yes
	IPV6_PEERROUTES=yes
	USERCTL=no
	NM_CONTROLLED=no
	HWADDR=08:00:27:ff:d7:a0
	NETMASK=255.255.255.0
	GATEWAY=10.0.2.2
	DNS1=8.8.8.8
	DNS2=8.8.4.4
```

- Restart network service:
```
	systemctl restart network
```

### Open Ports
- firewall-cmd --get-active-zones
- sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent
- firewall-cmd --reload

### Putty to VirtualBox CentOS 7 Minimal
- [Ref: http://ask.xmodulo.com/access-nat-guest-from-host-virtualbox.html](http://ask.xmodulo.com/access-nat-guest-from-host-virtualbox.html)
- Define port forwarding at VirtualBox
```
	Protocol : TCP
	Host IP : 127.0.0.1
	Host Port : 2290
	Guest IP : 10.0.2.15
	Guest Port : 22
```
### VirtualBoxGuestAdditions

#### Update
- yum update
- yum groupinstall “Development Tools”
- reboot

#### Mount
- from virtual box menu “Devices” insert guest addition iso dvd
```
mkdir /media/VirtualBoxGuestAdditions
mount -r /dev/cdrom /media/VirtualBoxGuestAdditions
```
- On CentOS/Red Hat (RHEL) 7/6/5, EPEL repo is needed
```
rpm -Uvh http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-8.noarch.rpm
```
- Fedora 21/20/19/18/17/16/15/14/13/12, CentOS/RHEL 7/6/5 ##
```
yum install gcc kernel-devel kernel-headers dkms make bzip2 perl
```
- Current running kernel on Fedora, CentOS 7/6 and Red Hat (RHEL) 7/6 
```
KERN_DIR=/usr/src/kernels/`uname -r, 
```
- Export KERN_DIR ##
```
export KERN_DIR
```

#### Install VirtualBoxGuestAdditions
```
cd /media/VirtualBoxGuestAdditions
```
- 32-bit and 64-bit systems run following
```
./VBoxLinuxAdditions.run
```
- Reboot

#### Shared Folder (not tested on Centos 7)
Usually resides at mount point /media/sf_<shared folders name>
Virtualbox Shared Folder. Shared folder is owned by group named 'vboxsf'
- add user to vboxsf
```
sudo usermod -aG vboxsf $(whoami)
```
- reboot
- try logging in