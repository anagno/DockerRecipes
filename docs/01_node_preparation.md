# Node preparation

Before even starting with setting up the cluster, we should prepare the 
nodes. These steps should be repeated for all the nodes of the cluster.

## Operating system 

The most simple OS to use is [Ubuntu](https://ubuntu.com/download/raspberry-pi). 
You should use a 64bit image, because generally many of the images we are using are 
only available for aarch64 not the arm4vl (which is the 32 bit version).
All the rest of the instructions are using an ubuntu os.

## Set up the hostname

Set up the hostname of the new node by typing it in the hostname file

```
sudo nano /etc/hostname 
```

## Enabling the fans

The raspberry pi 4 have the tendancy to get warm. So it is wise to have a fan. I am
using PWN fans, so I can enable the fans based on the temperature. But to achieve that
we need to install and compile some libraries.

```
sudo apt-get install build-essential
cd
wget http://www.airspayce.com/mikem/bcm2835/bcm2835-1.68.tar.gz
tar zxvf bcm2835-1.68.tar.gz
cd bcm2835-1.68
./configure
make
sudo make install
cd
sudo apt install -y git
git clone https://gist.github.com/1c13096c4cd675f38405702e89e0c536.git
cd 1c13096c4cd675f38405702e89e0c536
make
sudo make install
```

!!! note
    To print the current temperature of the node, we can use
    ```
    cpu=$(</sys/class/thermal/thermal_zone0/temp)
    echo "$((cpu/1000)) c"
    ```

## Enable cgroups limit support

For better control of the resources of the nodes we need to activate the cgroups

```
# Append the cgroups and swap options to the kernel command line
# Note the space before "cgroup_enable=cpuset", to add a space after the last existing item on the line
sudo sed -i '$ s/$/ cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1 swapaccount=1/' /boot/firmware/cmdline.txt
```


## Update the node

Update and upgrade the node and restart

```
sudo apt-get update && sudo apt-get dist-upgrade
sudo reboot
```

## Install and configure Docker 

```
sudo apt install -y docker.io

# Create or replace the contents of /etc/docker/daemon.json to enable the systemd cgroup driver
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF


cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system

sudo systemctl enable docker.service
sudo docker info
sudo reboot
```

## Install kubernetes

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

# Add the Kubernetes repo
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt update && sudo apt install -y kubelet kubeadm kubectl

# Disable (mark as held) updates for the Kubernetes packages
sudo apt-mark hold kubelet kubeadm kubectl

```

!!! note
    At time of writing, Ubuntu 16.04 (Xenial Xerus ) Kubernetes repository was available 
    but in future when the kubernetes repository is available for Ubuntu 20.04 then 
    replace xenial with focal word in above ‘apt-add-repository’ command.


## Resources

* https://opensource.com/article/20/6/kubernetes-raspberry-pi
* https://askubuntu.com/questions/1189480/raspberry-pi-4-ubuntu-19-10-cannot-enable-cgroup-memory-at-boostrap
* https://github.com/kubernetes/kubeadm/blob/master/docs/ha-considerations.md#kube-vip
* https://thebsdbox.co.uk/2020/01/02/Designing-Building-HA-bare-metal-Kubernetes-cluster/#Networking-load-balancing
* https://tanzu.vmware.com/content/blog/exploring-kube-apiserver-load-balancers-for-on-premises-kubernetes-clusters
* https://icicimov.github.io/blog/kubernetes/Kubernetes-cluster-step-by-step-Part5/
* https://stackoverflow.com/questions/56634139/how-to-use-the-master-node-as-worker-node-in-kubernetes-cluster
* https://www.shogan.co.uk/kubernetes/building-a-pi-kubernetes-cluster-part-3-worker-nodes-and-metallb/
* https://opensource.com/article/20/7/homelab-metallb


* https://blog.lzoledziewski.pl/2020/06/01/noctua-5v-pwm-fan-rpi-4/
* https://blog.driftking.tw/en/2019/11/Using-Raspberry-Pi-to-Control-a-PWM-Fan-and-Monitor-its-Speed/
* https://www.raspberrypi.org/forums/viewtopic.php?f=63&t=244194
* https://gist.github.com/alwynallan/1c13096c4cd675f38405702e89e0c536