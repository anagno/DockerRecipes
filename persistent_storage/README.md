# Introduction

[Ceph](https://ceph.com/) is a distributed file storage system,
that we will use for the our Docker Swarm. 

Unfortunately docker does not support yet 
[priviliged containers](https://github.com/docker/swarmkit/issues/1030), 
which are needed for the Bluestore in the Ceph. Until this 
is resolved we will not be able to deploy it in the swarm and we will 
have to deploy them manually to each node. 


!> **Important** When the problem with the privileged containers is 
resolved, I will have to rethink my approach. Currently I am also 
using the Ceph nodes as docker nodes. The ceph/docker nodes do not 
mount the Ceph filesystem. So they are capable of running services 
that do not store any state. In case of a catastrophic failure
(e.g. power outage), the docker nodes that mount the Ceph filesystem
expect the filesystem to be available. So I have to first to start 
the Ceph nodes and then the Docker nodes. If the Ceph cluster is not 
available the docker nodes will fail to start. So I will have to be 
careful with the coupling of the services. So If ever decide to 
create a stack for the Ceph cluster, this has to be considered. 
Furthermore, the update of the Ceph cluster is of paramount importance.
You simply can not update all the nodes at the same time, because 
probably the cluster will fail to start. I will have to find a way
of running a version to a specific version to a number of nodes,
and if everything is ok, then move the rest of the nodes. All this 
consideration must be taken into account, otherwise the cluster will
become no functional and data loss is guaranteed. 

A minimal system will have at least one Ceph monitor (ceph-mon), 
Ceph Manager (ceph-mgr) and three Ceph OSDs (ceph-osd) for peering 
and object durability. The Ceph Metadata Server (ceph-mds) is also 
required when running Ceph Filesystem clients, like in the case 
we want to use our Ceph cluster in Docker. 

The easiest way of deploy a Ceph cluster in nodes is through docker.
Ceph provides docker [images](https://hub.docker.com/r/ceph/daemon/)
that containes the necessary daemons to deploy the cluster.

?> **Note** I tried a couple of ways of deploying the cluster. The 
documentation mentions that the easiest way to deploy as Ceph cluster 
is by using the ceph-deploy 
[CLI](http://docs.ceph.com/docs/mimic/start/). 
But when I tried to use the CLI to deploy the cluster I received 
some messages that the necessary applications were not available.
For the moment (i.e. 3/10/2018), the problem is that there is no port
[for the Raspberry Pi](http://lists.ceph.com/pipermail/ceph-users-ceph.com/2017-June/018673.html).

Furthermore, currently the docker images has a couple of 
[problems](https://github.com/ceph/ceph-container/issues/1184)
and some necessary tools have not been integrated yet to the latest
[release](http://docs.ceph.com/docs/master/releases/schedule/).

The docker images are only available for aarch64, so we need a 
compatible image. I have already presented how you can create a 
compatible 64 bit image for the boards/computers.

Finally, in my local network I have a dedicated dns server, so I 
do not have to change the /etc/hostfiles or other similar tricks, in the 
local hosts. Moreover for the Ceph cluster we need an NTP server 
to prevent issues arising from clock drift. So make sure you have 
installed it in all servers.

?> **Tip** The setup/update order of the nodes should be:
1) ceph-mon
2) ceph-mgr
3) ceph-osd
4) ceph-mds


# Resources

* http://docs.ceph.com/docs/mimic/start/intro/
* https://blog.raveland.org/post/raspian_ceph_en/
* https://geek-cookbook.funkypenguin.co.nz/ha-docker-swarm/shared-storage-ceph/#setup-osds
* http://docs.ceph.com/docs/
* https://opensource.com/business/15/7/running-ceph-inside-docker
* http://blog.ruanbekker.com/blog/2018/06/13/setup-a-3-node-ceph-storage-cluster-on-ubuntu-16/
* https://hub.docker.com/r/ceph/daemon/
* https://github.com/ceph/ceph-container/tree/master/src/daemon
* https://github.com/sepich/ceph-swarm

