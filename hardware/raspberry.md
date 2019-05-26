# Raspberry Pi

As already mentioned, for the Ceph library we need an 64 bit operating system.
Generally many of the images we are using are only available for aarch64 not 
the arm4vl (which is the 32 bit version). So it is better to install an aarch64
image. For that we will have to create our own images for the Raspberry Pi.

Rasbian does not have an official aarch64 image at the moment of writting this
(i.e. 26/05/2019). I tried multiple distros, but debian was the most stable and 
I could install correctly the ceph-common package and docker.


## Compiling a kernel and create a custom image

The easiest way of creating a custom Debian image for the Raspberry Pi is by 
using the scripts from [drtyhlpr](https://github.com/drtyhlpr). 


So open a terminal and start typing:

```bash
git clone https://github.com/drtyhlpr/rpi23-gen-image.git

# The dependencies mention in the readme page of the 
# rpi23-gen-image project
sudo apt-get intall debootstrap debian-archive-keyring qemu-user-static binfmt-support dosfstools rsync bmap-tools whois git bc psmisc dbus sudo

cd rpi23-gen-image
```

Now that we have downloaded the scripts, we need a template parameter file.
Here are mine:

* [Raspberry Pi 3 model B](_assets/rpi3-with-ceph ':ignore')
* [Raspberry Pi 3 model B+](_assets/rpi3P-with-ceph ':ignore')
* [Linux kernel configuration](_assets/linux.config ':ignore')

Download the files, copy them in the 'templates' folder and just execute:

```bash
sudo CONFIG_TEMPLATE=rpi3-with-ceph ./rpi23-gen-image.sh
```

Altertanitive you can activate the menuconfig of the kernel by using the 
`KERNEL_MENUCONFIG=true` parameter in the parameter file. 
The extra modules we need to activate are:

* CEPH_FS
* CEPH_FSCACHE
* CEPH_FS_POSIX_ACL
* CEPH_LIB
* CEPH_LIB_USE_DNS_RESOLVER
* CONFIG_CGROUP_NET_PRIO
* CONFIG_NF_NAT_IPV4
* CONFIG_IP_NF_TARGET_MASQUERADE
* CONFIG_NF_NAT
* CONFIG_NF_NAT_NEEDED
* CONFIG_IPVLAN (optional)
* CONFIG_RT_GROUP_SCHED (optional)
* CONFIG_CFS_BANDWIDTH (optional)
* CONFIG_CGROUP_HUGETLB (optional)
* CONFIG_CGROUP_PERF (optional)
* CONFIG_MEMCG_SWAP_ENABLED (optional)
* CONFIG_MEMCG_SWAP (optional)
* CONFIG_CGROUP_PIDS (optional)
* CONFIG_IP_NF_TARGET_REDIRECT (optional)
* CONFIG_AUFS_FS (optional)

?> **Tip** To search in the menuconfig just press "/".

To check if the kernel of a computer is compatible with docker, the
docker team has released a 
[script](https://docs.docker.com/install/linux/linux-postinstall/#kernel-compatibility).

You can execute it by running:

```bash
curl https://raw.githubusercontent.com/docker/docker/master/contrib/check-config.sh > check-config.sh
chmod 777 check-config.sh 
./check-config.sh
```
The extra modules with the "(optional)" tag in the list are not strictly necessary, 
but they are there for the script.

Once the script is finished you will find your image under the images folder.
Write the image in an sd card and boot your Raspberry Pi.


## Setting up after first boot

After we install our operating system in the Raspberry Pi, we still have to make
some small changes. So log in the Rasbperry Pi by using SSH and execute:

```bash
sudo apt-get update && sudo apt-get upgrade

# The first thing that I usually do is set the hostname of our new node
sudo nano /etc/hostname 
sudo nano /etc/hosts

sudo nano /boot/cmdline.txt 
# add cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1 
# in the end of the line
# https://www.raspberrypi.org/forums/viewtopic.php?t=203128
# This is for limiting the memory consumption of some containers
# and not only

# change the password of the Raspberry Pi
# I usually use the apg to generate random passwards
passwd 
sudo systemctl reboot
```

## Automating completely the creation of the image

By creating a `custom.d` folder in the same folder with the `rpi23-gen-image.sh`
you can use your own scripts that will be executed before the image is packed.
My script looks like:

```
. ./functions.sh

# Mounting the ceph fs
chroot_exec apt-get -qq -y update 
chroot_exec apt-get -qq -y install ceph-common

mkdir -p "${R}/mnt/"
mkdir -p "${R}/mnt/storage/"
echo "IP_1:6789,IP_2:6789,IP_3:6789:/ /mnt/storage ceph name=swarm,secret=YOUR_SECRET,noatime,_netdev 0 0" >> "${ETC_DIR}/fstab"

# Installing docker
# There is a bug in the iptables 
# https://www.raspberrypi.org/forums/viewtopic.php?t=228261
chroot_exec update-alternatives --set iptables /usr/sbin/iptables-legacy

echo "deb [arch=arm64] https://download.docker.com/linux/debian buster stable" >> "${ETC_DIR}/apt/sources.list"
echo "# deb-src [arch=arm64] https://download.docker.com/linux/debian buster stable" >> "${ETC_DIR}/apt/sources.list"

curl -fsSL https://download.docker.com/linux/debian/gpg | chroot_exec apt-key add -

chroot_exec apt-get -qq -y update
chroot_exec apt-get -qq -y install docker-ce docker-ce-cli containerd.io


# https://www.raspberrypi.org/forums/viewtopic.php?t=203128
# This is for limiting the memory consumption of some containers
# and not only

sed -i " 1 s/.*/& cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1/" "${BOOT_DIR}/cmdline.txt"
```

## Memorable mentions

The [HypriotOS](https://blog.hypriot.com) is an image with docker 
pre-installed. But as already mentioned it is better to have an 64 bit 
operating system. The 
[official image](https://github.com/hypriot/image-builder-rpi) 
does not support yet the 64bit operating system. There is an 
[unofficial image](https://github.com/DieterReuter/image-builder-rpi64).
The HypriotOS is really nice project, but the provided kernels with 
the images do not support Ceph. So I really had to compile my own
kernels.


## Resources 
* https://withblue.ink/2017/12/31/yes-you-can-run-docker-on-raspbian.html
* https://www.marksei.com/docker-on-raspberry-pi-raspbian/
* https://github.com/drtyhlpr/rpi23-gen-image
* https://wiki.debian.org/RaspberryPi3
