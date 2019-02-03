# Introduction

From the [documentation](http://docs.ceph.com/docs/mimic/start/intro/):
A Ceph OSD (object storage daemon, ceph-osd) stores data, handles data 
replication, recovery, rebalancing, and provides some monitoring information 
to Ceph Monitors and Managers by checking other Ceph OSD Daemons for a 
heartbeat. At least 3 Ceph OSDs are normally required for redundancy and 
high availability.

Before being able to set up ceph-osd node you need to set up a
ceph-mon node. But you can remove it afterwards once you set up the 
ceph-osd node.

?> **Note** TODO: Find out if we can remove this limitation

After you set up the ceph-mon node you will have to retrieve the necessary
encryption keys. So execute:

```bash
docker exec ceph-mon ceph auth get client.bootstrap-osd -o /var/lib/ceph/bootstrap-osd/ceph.keyring
```

?> **Note** After synchronizing the keys, you can remove the remove the 
ceph-mon container. Do that by executing '''docker container rm ceph-mon'''

Before initializing the disk, we will have to destroy the partition 
tables of the disk. So make sure, that the disk does not contain 
anything usefull and execute:

```bash
docker run -d --privileged=true \
-v /dev/:/dev/ \
-e OSD_DEVICE=/dev/sda \
ceph/daemon:v3.1.0-stable-3.1-mimic-centos-7-aarch64 zap_device

docker run -d --net=host \
--memory="1800M" \
--privileged=true \
--pid=host \
--name="ceph-osd" \
--restart=always \
-v /etc/ceph:/etc/ceph \
-v /var/lib/ceph/:/var/lib/ceph/ \
-v /dev/:/dev/ \
-e OSD_DEVICE=/dev/sda \
-e OSD_TYPE=disk \
-e OSD_BLUESTORE=1 \
ceph/daemon:v3.1.0-stable-3.1-mimic-centos-7-aarch64 osd_ceph_disk
```

!> **Note** The OSD_TYPE=directory in the ceph-osd has many limitations. 
Mainly I had problems with the lenght of files the osd node was creating.
So it is much easier to use the default options and use a hole external
drive as the osd storage. For that puprpose just make sure you are
mounting your external hard drive on boot.

!> **Note** During the deployment of the container I am also setting 
some limitations to the amount of the Ram of the memory it can use.
The reason behind that is, that ceph-osd needs a lot of memory Ram.
The amount of memory they need is propotional to the sizes of the disks.
Some of the options we set on the ceph.conf files are exactly for that
reason. But still some times, the usage was getting out of control.
When the operating system is running low on memory, the kernel
is starting to kill processes to free memory. This can lead, to 
important services dying. So I am also instructing the docker of 
not giving more memory Ram to the container. So adapt the value
depending on the board you are using.