# Armbian compatible boards

My new favorite boards are currently:

* [Libre Computer Board ROC-RK3328-CC](https://libre.computer/products/boards/roc-rk3328-cc/) 
with 4GB Ram
* [PINE64 ROCK64](https://www.pine64.org/?page_id=7147) with 4GB Ram

To be frank, both of them are quite similar (and I could say almost identical).
Raspberry Pi is very limiting for this project, because it does not have a lot of Ram.
Furthermore it does not have an official 64 bit image, and creating one is very difficult.
So I was searching for some alternatives, and the [armbian](https://www.armbian.com/)
project came to the rescue.  

The armbian project has a plethora of supported boards, and the ones I choose were
the above afformentioned. When choosing a board, make sure that the architecture of 
the SOC is suitable for your needs. This project needs a 64 bit operating system, 
so I made sure that the boards have a 64 bit capable SOC.


## Compiling a kernel and create a custom image

The easiest way of creating a custom images is by using the scripts 
provided by [armbian](https://github.com/armbian/build). 

The instructions are quite clear and I did not really have a problem
compiling my kernels. Again activate the same modules:

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


I used as a base system Ubuntu, because I am more familiar, but also
the other systems will not have any problems.

## Setting up after first boot

Similar to the Rasbperry Pi installation, after we install our 
operating system in the board, we still have to make
some small changes. So log in the board by using SSH and execute:

```bash
sudo apt-get update && sudo apt-get upgrade
sudo nano /etc/hostname 
passwd 
sudo passwd -l root
sudo reboot
```


## Resources 
* https://github.com/armbian/build
* https://docs.armbian.com/Developer-Guide_User-Configurations/#user-provided-image-customization-script