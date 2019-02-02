# Hardware

This project started as an attempt to use only Raspberry Pis 3. If you are
really a patient man, you might be able to achieve it, but the reality is 
that they are not enough powerfull enough. Furthermore the support is really
limited. Although Raspberry Pi has a 64 bit processor (i.e. ARM Cortex A-53), 
there is not yet an official 64 bit operating system. So you will have to create 
your own images and compile your own kernel. 

Technically you can install and use docker in a 32 bit operating system, but 
I would not suggest it. For example the Ceph library (details coming soon :) ) 
we need a 64 bit operating system, because the official images do not support
32 bit ARM devices. 

Most of the official images do not really support 
ARM 32 bit operating systems, and you will have to build and create you own 
images. Some projects (e.g. Gitlab) do not even support ARM processors. 
So my suggestion will be to compile your own image/kernel.

The hardware I am currently using is:

* 5 x [Raspberry Pi 3 Model B](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/)
* 5 x [Raspberry Pi 3 Model B+](https://www.raspberrypi.org/products/raspberry-pi-3-model-b-plus/)
* 2 x [ODROID-C2](https://wiki.odroid.com/odroid-c2/odroid-c2) with 2GB RAM
* 2 x [Libre Computer Board ROC-RK3328-CC](https://libre.computer/products/boards/roc-rk3328-cc/) 
with 4GB Ram
* 3 x [PINE64 ROCK64](https://www.pine64.org/?page_id=7147) with 4GB Ram
* 4 x [Raspberry Pi X830 V2.0 HDD Expansion Board and 3.5" SATA HDD Storage Modules for Raspberry Pi 3 B+Plus/3B](https://geekworm.com/products/raspberry-pi-x830-3-5-sata-hdd-board-24v-2a-power-supply-metal-case-kit)
* 2 x [Raspberry Pi X830 V2.0 HDD Expansion Board and 3.5" SATA HDD Storage Module for Raspberry Pi 3 B+Plus/3B](https://geekworm.com/products/x820-v3-0-usb-3-0-2-5-inch-sata-hdd-ssd-storage-expansion-board?variant=17470073372730)
* 2 x [Nextcloud boxes](https://nextcloud.com/box/). Unfortunatelly the 1 TB USB3 
hard drive from WDLabs (which is used in the nextcloud box) is no longer available.
WD stop making, and that is why I switched to Geekworm cases, which in my personal 
opinion are much better and flexible. 
* 2 x [ZOTAC ZBOX BI324 BAREBONE INTEL N306](https://www.zotac.com/lk/product/mini_pcs/zbox-bi324) with all the necessary
hardware (i.e. Ram and hard disk which are not included).
* 1 x [Intel NUC7I3DNK2E](https://ark.intel.com/products/122495/Intel-NUC-Kit-NUC7i3DNKE) and again with all
the necessary hardware (i.e. Ram and hard disk which are not included).

My current setup is a bit of an overkill. If I had to redo it from the begging I 
would not buy Raspberry Pis. I would only buy:
* PINE64 ROCK64 or Libre Computer Boards ROC-RK3328-CC (depending on what you prefer)
and the 
* Zotac barebones (or the cheapest x86 barebone pc I could find). 

The ARM  boards I would use them for storage nodes for the ceph cluster (and of 
course as nodes in the docker cluster, but the will not have persistent storage -- 
details later) and the x86 barbones pc as nodes for the docker (which will connect 
to the ceph cluster). Of course you will need plenty sd cards for the boards and a 
way to stack them.

I will not describe on how to set up DHCP, DNS, e.t.c servers because they are out of
the scope of this project. Before starting I would suggest having a good convention 
of naming your devices, because when (not not if -- do not doubt about the fact that 
hardware will fail) the hardware fails it will be difficult to locate without good 
naming conventions. I would also suggest of not hardcoding IPs, because if you have 
to scale your network and everything is hardcoded, then you are out of luck.

I would not suggest setting up the DHCP, DNS server on the docker, because you will 
have a chicken and egg problem. If there is a total failure (e.g. a power outage)
you will not be able to restore the cluster. If the DHCP, DNS server is inside the 
cluster, then the services will not be able to start because the cluster is not up, 
and the cluster is not up, because there is no DHCP, DNS server. Currently I am using
a DD-Wrt router for these services. In case the router fails, I always have on stand
by an another router, that I can set up with the same parameters. Of course they 
will be a downtime, but you have to choose your fights.