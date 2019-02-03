# Raspberry Pi

As already mentioned, for the Ceph library we need an 64 bit operating system.
Generally many of the images we are using are only available for aarch64 not 
the arm4vl (which is the 32 bit version). So it is better to install an aarch64
image. For that we will have to create our own images for the Raspberry Pi.

Rasbian does not have an official aarch64 image at the moment of writting this
(i.e. 03/02/2019). I tried multiple distros, but debian was the most stable and 
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

Download the file and just execute (do not forget to copy the parameter files
in your working folder):

```bash
sudo CONFIG_TEMPLATE=rpi3-stretch-with-ceph ./rpi23-gen-image.sh
```

The parameter files will also activate the menuconfig of the kernel because we
need to activate some extra modules. The modules we need are:

* CEPH_FS
* CEPH_FSCACHE
* CEPH_FS_POSIX_ACL
* CEPH_LIB
* CEPH_LIB_USE_DNS_RESOLVER
* CONFIG_CGROUP_NET_PRIO
* CONFIG_IPVLAN (optinoal)
* CONFIG_RT_GROUP_SCHED (optinoal)
* CONFIG_CFS_BANDWIDTH (optinoal)
* CONFIG_CGROUP_HUGETLB (optinoal)
* CONFIG_CGROUP_PERF (optinoal)
* CONFIG_MEMCG_SWAP_ENABLED (optinoal)
* CONFIG_MEMCG_SWAP (optinoal)
* CONFIG_CGROUP_PIDS (optinoal)

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

## Booting from a hard-drive

This step is **not** really necessary, but for full documentation I leave it.
Especially if you want to have an osd node I would not suggeste to boot from a
hard-drive. It is easier to just use the whole drive. 
So move to the **next sections**.

The image we are using is a much more basic one we will have to adjust some 
file to be able to 
[boot](https://www.maketecheasier.com/boot-up-raspberry-pi-3-external-hard-disk/) 
from an 
[external hard drive](https://github.com/hypriot/image-builder-rpi/issues/260)

To boot from an external hard drive adjust the root parameter in 
the /boot/cmdline.txt to point to the hard drive:

```
dwc_otg.lpm_enable=0 console=tty1 root=/dev/sda2 rootfstype=ext4 cgroup_enable=cpuset cgroup_memory=1 swapaccount=1 elevator=deadline fsck.repair=yes rootwait console=ttyAMA0,115200 net.ifnames=0 rootdelay=5
```

and afterwards adjust the /etc/fstab to point again to the hard drive:

```
proc /proc proc defaults 0 0
/dev/sda1 /boot vfat defaults 0 0
/dev/sda2 / ext4 defaults,noatime 0 1
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