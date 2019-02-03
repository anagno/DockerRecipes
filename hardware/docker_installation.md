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

sudo apt-get install curl
sudo curl -fsSL get.docker.com -o get-docker.sh && sh get-docker.sh
```

## Joining the swarm 

?> **Tip** Read first the [Swarm mode]() section

Just execute in each node:

```
docker swarm join --token (token_received_from_manager)
```

!> **Important**  Use the correct token provided from your token manager.