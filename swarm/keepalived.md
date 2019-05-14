# Introduction

In order to provide seamless external access to clustered resources, regardless 
of which node they're on and tolerant of node failure, you need to present a single 
IP to the world for external access. That is why we 
use [keepalive](https://keepalived.org/).


# Service mode

Run the following on the primary:

```bash
sudo apt-get install keepalived
sudo nano /etc/keepalived/keepalived.conf
```

```
vrrp_instance VRRP1 {
    state MASTER
    # Specify the network interface to which the virtual address is assigned
    interface eth0
    # The virtual router ID must be unique to each VRRP instance that you define
    virtual_router_id 51
    priority 200
    advert_int 1
    authentication {
        auth_type PASS
        # Change password
        auth_pass YOURSECRETPASSWORD
    }
    virtual_ipaddress {
        # Change the ip
        192.168.1.100/24
    }
}
```
 
Run the following on the secondaries:

```bash
sudo apt-get install keepalived
sudo nano /etc/keepalived/keepalived.conf
```

```
vrrp_instance VRRP1 {
    state BACKUP
    # Specify the network interface to which the virtual address is assigned
    interface eth0  #enxb827ebd24cfa
    # The virtual router ID must be unique to each VRRP instance that you define
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        # Change password
        auth_pass YOURSECRETPASSWORD
    }
    virtual_ipaddress {
        # Change the ip
        192.168.1.100/24
    }
}
```

# Docker way

There is no official image, but the "most official" image can be found
[here](https://github.com/osixia/docker-keepalived).
Unfortunatelly it does not provided an arm image.

!> **Important** This is untested. 


Run the following on the primary:

```bash
docker run -d \
  --name keepalived \
  --restart=always \
  --cap-add=NET_ADMIN --net=host \
  -e KEEPALIVED_UNICAST_PEERS="#PYTHON2BASH:['local1.lan', 'local2.lan', 'local3.lan', 'local4.lan']" \
  -e KEEPALIVED_VIRTUAL_IPS=192.168.1.100 \
  -e KEEPALIVED_PRIORITY=200 \
  osixia/keepalived:2.0.10 
```

Run the following on the secondaries:

```bash
docker run -d --name keepalived --restart=always \
  --cap-add=NET_ADMIN --net=host \
  -e KEEPALIVED_UNICAST_PEERS="#PYTHON2BASH:['local1.lan', 'local2.lan', 'local3.lan', 'local4.lan']" \
  -e KEEPALIVED_VIRTUAL_IPS=192.168.1.100 \
  ***-e KEEPALIVED_PRIORITY=100 \***
  osixia/keepalived:2.0.10
```


# Resources
* https://geek-cookbook.funkypenguin.co.nz/ha-docker-swarm/keepalived/
* https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/load_balancer_administration/ch-initial-setup-vsa
* https://docs.oracle.com/cd/E37670_01/E41138/html/section_uxg_lzh_nr.html
