# Introduction

From the [documentation](http://docs.ceph.com/docs/mimic/start/intro/):
A Ceph Monitor (ceph-mon) maintains maps of the cluster state, including
the monitor map, manager map, the OSD map, and the CRUSH map. These maps 
are critical cluster state required for Ceph daemons to coordinate with 
each other. Monitors are also responsible for managing authentication between
daemons and clients. At least **three** monitors are normally required for 
redundancy and high availability. I suggest to use five (because of the 
consensus protocols used, you can not just use 4. It is better to use 5.).

To deploy the first ceph-mon we just have to SSH to the node and execute:

```bash
docker run -d --net=host \
--restart always \
--name="ceph-mon" \
--privileged=true \
-v /etc/ceph:/etc/ceph \
-v /var/lib/ceph/:/var/lib/ceph/ \
-e MON_IP=(real_ip_of_node) \
-e CEPH_PUBLIC_NETWORK=(real_submask_of_node)/24 \
ceph/daemon:v3.1.0-stable-3.1-mimic-centos-7-aarch64 mon
```

!> **Important** Make sure to replace the MON_IP with the correct IP 
and the CEPH_PUBLIC_NETWORK with the correct submask.

?> **Note** I tried to not use a hardcoded ip can but I did not managed
to make it work. The container does not support domain names. This is WIP.

The container is now creating the necessary configuration files for the 
ceph-mon under the ```/etc/ceph``` folder.

For the rest of the nodes, we have to copy **ALL** the contents 
the ```/etc/ceph``` folder from the first node to each new nodes, 
**before** executing the above command.

But before doing that let` adjust them to fit our purposes.
The most important one is ```ceph.conf```. I retrieve the ceph.conf
from one of my nodes:

```yaml
[global]
fsid = MY_FSID_THAT_WAS_REMOVED
mon initial members = HOSTNAME1, HOSTNAME2, HOSTNAME3
mon host = IP_1, IP_2, IP_3
public network = SUBNET
cluster network = SUBNET
osd journal size = 100
log file = /dev/null

[mon]
mgr initial modules = dashboard

# for rock64
[osd]
bluestore_cache_size = 343000000 #700MB
bluestore_cache_size_hdd = 343000000 #700MB
bluestore_cache_size_ssd = 343000000 #700MB
bluestore_cache_kv_max = 171500000 #700MB
osd map cache size = 500
osd map cache bl size = 25
osd map cache bl inc size = 50
osd map message max = 50
osd recovery delay start = 60
osd recovery max active = 3 # if it too much then change this to 2
osd recovery op priority = 2
osd client message size cap = 524288000 #500MB 
```

!> **Important** Make sure to replace the HOSTNAME1, HOSTNAME2, HOSTNAME3,
IP_1, IP_2, IP_3 and SUBNET with the correct values.
and the CEPH_PUBLIC_NETWORK with the correct submask.

The ```mon initial members = HOSTNAME1, HOSTNAME2, HOSTNAME3``` has to have
the initial members of our cluster. It does not have to have all the 
available ceph-mon nodes. These nodes are the initial nodes that 
will be available to "welcome" any new nodes. After the initiallization
of the new node, the nodes will exchange the necessary information and 
retrieve all the available nodes. These nodes just have to be available 
for the initial communication.

Afterwards I am adjusting some values of the resources the node is using.
Because I am not using a full blown computer, some parameters were too 
aggresive for the boards. So adjust them, to reduce the amount of recouses
being used in the nodes. If you do not do that step, then you will have 
crashes (especially in the ceph-osd nodes).

These settings are not necessary for the ceph-mon node, but they are important
for the rest of the services we have to deploy (especially when a node is 
running multiple services for ceph.)

!> **Important** The ceph.conf file is necessary for all services. I am 
describing it here, because ceph-mon is the first one using it.

In my example I mentioned already the parameters I used in the 
Libre Computer Board ROC-RK3328-CC/PINE64 ROCK64 boards.

In the Raspberry Pis I am using:

```yaml
[osd]
bluestore_cache_size = 1000000 #100MB 
bluestore_cache_size_hdd = 1000000 #100MB 
bluestore_cache_size_ssd = 1000000 #100MB 
bluestore_cache_kv_max = 500000 #100MB 
osd map cache size = 50
osd map cache bl size = 25
osd map cache bl inc size = 50
osd map message max = 25
osd recovery delay start = 60
osd recovery max active = 1
osd recovery op priority = 2
osd client message size cap = 104857600 #100MB 
```

and in the ODROID-C2 boards I am using:

```yaml
[osd]
bluestore_cache_size = 125000000 #500MB 
bluestore_cache_size_hdd = 125000000 #500MB 
bluestore_cache_size_ssd = 125000000 #500MB 
bluestore_cache_kv_max = 62500000 #500MB 
osd map cache size = 500
osd map cache bl size = 25
osd map cache bl inc size = 50
osd map message max = 50
osd recovery delay start = 60
osd recovery max active = 2
osd recovery op priority = 2
osd client message size cap = 524288000 #500MB 
```

After each deployment of a node we can check if the nodes were 
added succesfully by executing:

```
docker exec ceph-mon ceph status
```


# Resources
* https://github.com/sepich/ceph-swarm
* http://docs.ceph.com/docs/luminous/rados/configuration/bluestore-config-ref/#cache-size

