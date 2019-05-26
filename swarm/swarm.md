# Introduction


# Initializing the cluster

An docker should already be installed to the node. To one of the docker initialize the cluster

```
docker swarm init --advertise-addr 192.168.X.X
```

The ip is necessary when in the same node you have already set the keepalived client.

# Setting up a node for the cluster

?> **Tip** Read first the [Persistent storage]() section

Log in the Raspberry Pi by using the 

```bash
sudo apt-get install ceph-common
sudo mkdir /mnt/storage
sudo nano /etc/fstab
```

monitor1_IP:6789, monitor2_IP: 6789,monitor3_IP:6789:/ /mnt/storage ceph name=swarm,secret=MY_SECRET,noatime,_netdev 0 0

In any docker manager execute:

```
docker node update --label-add persistence=ceph NAME_OF_NEW_NODE

docker node ls -q | xargs docker node inspect \
  -f '{{ .ID }} [{{ .Description.Hostname }}]: {{ .Spec.Labels }}'

docker node ls -q | xargs docker node inspect \
  -f '{{ .ID }} [{{ .Description.Hostname }}]: {{ .Description.Platform }}'
```

# Resources
* https://geek-cookbook.funkypenguin.co.nz/ha-docker-swarm/docker-swarm-mode/