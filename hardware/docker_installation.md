# Installing Docker on the computer

 The installation of the docker I use, is the one provided by docker.
 Docker provides a [script](https://github.com/docker/docker-install)
 for the automatic installtion of the docker. Just use it.

!> **Important**  There is a bug in the iptables of the Raspberry Pi.
The provided code takes it into account. You only have to change the
iptables in the Rasbperry Pis and not in the other boards. 

So just execute in each node:

```
# There is a bug in the iptables 
# https://www.raspberrypi.org/forums/viewtopic.php?t=228261
sudo update-alternatives --config iptables
# and choose iptables-legacy

# https://docs.docker.com/install/linux/docker-ce/debian/
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg2 \
    software-properties-common

curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -

# Change your arch to match your machine
sudo add-apt-repository \
   "deb [arch=arm64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

## Joining the swarm 

?> **Tip** Read first the [Swarm mode]() section

Just execute in each node:

```
docker swarm join --token (token_received_from_manager)
```

!> **Important**  Use the correct token provided from your token manager.


## Mounting the ceph file system

?> **Tip** See [mounting_clients]() section





## Resources 
* https://docs.docker.com/install/linux/docker-ce/debian/