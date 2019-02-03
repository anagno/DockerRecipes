# Introduction

From the [documentation](http://docs.ceph.com/docs/mimic/start/intro/):
A Ceph Metadata Server (MDS, ceph-mds) stores metadata on behalf of
the Ceph Filesystem (i.e., Ceph Block Devices and Ceph Object Storage
do not use MDS). Ceph Metadata Servers allow POSIX file system users to 
execute basic commands (like ls, find, etc.) without placing an enormous 
burden on the Ceph Storage Cluster.

To deploy a ceph-mds we execute:

```bash
docker run -d --net=host \
--name ceph-mds \
--restart always \
-v /var/lib/ceph/:/var/lib/ceph/ \
-v /etc/ceph:/etc/ceph \
-e CEPHFS_CREATE=1 \
-e CEPHFS_NAME=storage \
-e CEPHFS_DATA_POOL_PG=64 \
-e CEPHFS_METADATA_POOL_PG=64 \
ceph/daemon:v3.1.0-stable-3.1-mimic-centos-7-aarch64 mds
```
!> **Note** Here we are creating a filesystem called "storage".
We are setting the PG number relative to a small value. But it is 
always to increase the PG number but is IMPOSIBLE to reduce it.
So use a small number in the beggining and afterwards increase it.

You can increase the number of the PG by executing in one ceph-osd node:

```
docker exec ceph-mds ceph osd pool set storage_data pg_num 128
docker exec ceph-mds ceph osd pool set storage_data pgp_num 128

docker exec ceph-mds ceph osd pool set storage_metadata pg_num 64
docker exec ceph-mds ceph osd pool set storage_metadata pgp_num 64
```

To increase the number of replicas for you data execute:

```
docker exec ceph-mds ceph osd pool set storage_data size 3
docker exec ceph-mds ceph osd pool set storage_metadata size 4
```

To increase the active mds nodes in the cluster execute:

```
docker exec ceph-mds ceph tell mon.* injectargs --mon-allow-pool-delete=false
```


# Resources
* https://blog.zhaw.ch/icclab/deploy-ceph-troubleshooting-part-23/
* http://docs.ceph.com/docs/master/rados/operations/placement-groups/
* https://ekuric.wordpress.com/2016/02/09/increase-number-of-pgpgp-for-ceph-cluster-ceph-error-too-few-pgs-per-osd/
* https://ceph.com/planet/cephfs-ideal-pg-ratio-between-metadata-and-data-pools/
* http://docs.ceph.com/docs/master/cephfs/multimds/
