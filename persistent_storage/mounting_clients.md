# Mounting the CephFS on the clients 

Unfortunatelly, ceph is not directly supported as a remote volume
in the docker, so what I do is mounting the remote CephFS in the same
location for each node (e.g. ```/mnt/storage```) and assign a label to 
each of the nodes, to distinguish them from nodes that can not mount 
the CephFS. 


# Creating special credentials for the swarm

In order to mount the created CephFS, will need to provided authentication 
credentials. So we will have to create credentials for the docker nodes. To do 
that on one of the ceph-mon node execute:

```bash
docker exec ceph-mon ceph auth get-or-create client.swarm mon 'allow r' mds 'allow rw' osd 'allow * pool=storage'

docker exec ceph-mon ceph auth caps client.swarm mon 'allow r' mds 'allow rw' osd 'allow rwx pool=storage_metadata,allow rwx pool=storage_data'

# If the above does not work/ or you need less restrictive permissions execute:
ceph-mon ceph auth caps client.swarm mds 'allow' mgr 'allow *' mon 'allow *' osd 'allow *'
```

The above will create a new user called 'swarm'. To retrieve the key which will be used 
to mount the data, execute:

```bash
docker exec ceph-mon ceph auth get client.swarm
```


!> **Important** With the current approach all nodes share the same credentials.
So if a node is compromised, we will have to replace the credentials to all nodes.
Furthermore the same ceph pool is shared across all services. For the moment, it 
will have to suffice, but eventually I will have to find a better and more 
secure solution.

?> **Note** There are some  
[plugins] (https://rexray.readthedocs.io/en/stable/user-guide/storage-providers/ceph/)
that might be able to support this functionality, but I could not 
make it work easily and they would introduce the same dependencies.
So I prefered to install the clients directly on the operating system.

# Installing ceph-common and mounting the storage on a node

In each node execute:

```bash
sudo apt-get install ceph-common
sudo mkdir /mnt/storage
sudo nano /etc/fstab
```
Now add the below in the ```fstab``` file:

```
IP_1:6789,IP_2:6789,IP_3:6789:/ /mnt/storage ceph name=swarm,secret=YOUR_SECRET,noatime,_netdev 0 0
```

To test if everything works just execute:

```bash
sudo mount -a
```

and the CephFS will me mounted under ```/mnt/storage```.

!> **Important** Replace the IP_1,IP_2,IP_3 with the correct IPs/domain names of the ceph-mon nodes 
and YOUR_SECRET with the secret retrieved when you created the 'swarm' user.

?> **Note** To assign a label to the docker node, just execute from a manager: 
```docker node update --label-add persistence=ceph NAME_OF_NEW_NODE```



?> **Note** The default image of the Raspberry Pi does not include the necessary kernel modules.
That is the reason for compiling our own kernel.

# Resources
* https://github.com/ceph/ceph-container/issues/384
* https://keithtenzer.com/2017/03/30/openstack-swift-integration-with-ceph/#more-8840
* https://github.com/rexray/rexray/blob/master/.docs/user-guide/storage-providers/ceph.md#ceph-rbd
