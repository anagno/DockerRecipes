# Introduction

From the [documentation](http://docs.ceph.com/docs/mimic/start/intro/):
A Ceph Manager daemon (ceph-mgr) is responsible for keeping track of 
runtime metrics and the current state of the Ceph cluster, including 
storage utilization, current performance metrics, and system load. The 
Ceph Manager daemons also host python-based plugins to manage and expose 
Ceph cluster information, including a web-based dashboard and REST API. 
At least **two** managers are normally required for high availability.


Before deploying it, you will need the files from the 
```/etc/ceph``` folder. These files are necesarry. So again copy 
them from first node, to the node you are deploying.

To deploy a ceph-mgr execute:

```bash
docker run -d --net=host \
--restart=always \
--name="ceph-mgr" \
--privileged=true \
--pid=host \
-v /etc/ceph:/etc/ceph \
-v /var/lib/ceph/:/var/lib/ceph/ \
ceph/daemon:v3.1.0-stable-3.1-mimic-centos-7-aarch64 mgr
```

We can check if the nodes were added succesfully by executing:

```bash
docker exec ceph-mon ceph status
```


# Enabling the dashboard in the manager

Unfortunatelly this part of the deployment, I did not document.
The relative documentation is located 
[under](http://docs.ceph.com/docs/mimic/mgr/dashboard/#enabling)


I think what I did is that I execute: 

```bash
docker exec ceph-mgr ceph config-key put mgr/dashboard/server_addr ::
docker exec ceph-mgr ceph config-key put config/mgr/mgr/dashboard/server_port 7000
docker exec ceph-mgr ceph config-key put config/mgr/mgr/dashboard/ssl false
docker exec ceph-mgr ceph mgr module enable dashboard
```

To permanetly enable the dashboard, just add in the 
```/etc/ceph/ceph.conf``` :


```
[mon]
mgr initial modules = dashboard
```

!> **Note** We have already done that when we were configuring the ceph.conf

# Resources

* http://docs.ceph.com/docs/mimic/mgr/dashboard/
