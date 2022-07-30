# Apendix
These are instructions that were not fitting in any category

## Controloning the cluster from other computers

### Installing Ansible

Since I have a couple raspberries to manage, I will be using Ansible for that purpose.
So before we start, we have to install ansible for managing the nodes. 

The instructions from the [official page](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-ubuntu) are:

``` bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```

### Installing kubectl to manage the cluster

Just install ``kubectl`` to the other computer and copy the necessary files

``` bash
sudo snap install kubectl
# Copy the config file to the local computer in .kube folder
```

### Installing Helm for deploying services

Just follow the instructions from the [official page](https://helm.sh/docs/intro/install/#from-apt-debianubuntu)


## Decrypting kubectl secrets

If we want to see the secret we can use: 

```
kubectl -n longhorn-system get secret longhorn-crypto --template={{.data.CRYPTO_KEY_VALUE}} | base64 -d
kubectl -n authentication get secret openldap-admin -o jsonpath="{.data.admin-password}" | base64 -d
```


## Building the documentations

The documentation pages were build with [MkDocs](https://www.mkdocs.org/) 
and use the [Material](https://squidfunk.github.io/mkdocs-material/) theme

What you need to build them:

```
pip3 install mkdocs
pip3 install mkdocs-material
pip3 install mkdocs-same-dir
mkdocs serve
mkdocs build
```

## Listing the running containers on a node 

```
sudo k3s crictl ps
```

## Removing failed nodes from the etcd cluster

If a node fails it is not as simple to remove it and re-add it. Sometimes the failed node 
will remain part of the etcd cluster. So will we have first to remove it from the etcd cluster.


```bash
kubectl delete node failed_node
```
If the master nodes are tainted we will have to remove the taint for the moment to be able
to schedule a pod:

```bash
kubectl taint nodes other_master_node node-role.kubernetes.io/master=true:NoSchedule-
```

Then we can create a pod to manipulate the etcd cluster

```bash
# https://github.com/k3s-io/k3s/issues/2732#issuecomment-749484037
kubectl run --rm --tty --stdin --image quay.io/coreos/etcd:v3.5.1 etcdctl --overrides='{"apiVersion":"v1","kind":"Pod","spec":{"hostNetwork":true,"restartPolicy":"Never","securityContext":{"runAsUser":0,"runAsGroup":0},"containers":[{"command":["/bin/sh"],"image":"docker.io/rancher/coreos-etcd:v3.4.13-arm64","name":"etcdctl","stdin":true,"stdinOnce":true,"tty":true,"volumeMounts":[{"mountPath":"/var/lib/rancher","name":"var-lib-rancher"}]}],"volumes":[{"name":"var-lib-rancher","hostPath":{"path":"/var/lib/rancher","type":"Directory"}}],"nodeSelector":{"node-role.kubernetes.io/etcd":"true"}}}'
etcdctl --key /var/lib/rancher/k3s/server/tls/etcd/client.key --cert /var/lib/rancher/k3s/server/tls/etcd/client.crt --cacert /var/lib/rancher/k3s/server/tls/etcd/server-ca.crt member list
etcdctl --key /var/lib/rancher/k3s/server/tls/etcd/client.key --cert /var/lib/rancher/k3s/server/tls/etcd/client.crt --cacert /var/lib/rancher/k3s/server/tls/etcd/server-ca.crt member remove 1234567890ABCDEF
```

and the we re-taint the node

```bash
kubectl taint nodes other_master_node node-role.kubernetes.io/master=true:NoSchedule
```

Finally we can use the cluster_setup/setup_cluster.yml to re-populate the node.

!!!Note
    Remember to change the invetory if necessary


## Updating the raspberry pi firmware

If you have already an installed ubuntu on the Raspberry Pi you can execute:

```
sudo apt-get install rpi-eeprom

sudo rpi-eeprom-update
BCM2711 detected
Dedicated VL805 EEPROM detected
*** UPDATE AVAILABLE ***
BOOTLOADER: update available
CURRENT: Mon Jul 15 12:59:55 UTC 2019 (1563195595)
 LATEST: Thu Sep  3 12:11:43 UTC 2020 (1599135103)
 FW DIR: /lib/firmware/raspberrypi/bootloader/default
VL805: update available
CURRENT: 00013701
 LATEST: 000138a1

sudo rpi-eeprom-update -a
sudo reboot
```

or via ansible:

```
ansible all -b -m package -a "name=rpi-eeprom"
ansible all -b -m shell -a "rpi-eeprom-update"
ansible all -b -m shell -a "rpi-eeprom-update -a"
ansible all -b -m reboot
```

If you have not already set-up the Raspberry Pi, then we will have to
write first an sd card that will [update the firmware](https://rpi4cluster.com/k3s/k3s-nodes/#update-raspberry-pi-4-firmware)


## Enabling the fans

The raspberry pi 4 have the tendancy to get warm. So it is wise to have a fan. I am
using PWN fans, so I can enable the fans based on the temperature. But to achieve that
we need to install and compile some libraries.

For convienience I am already including under the folder node_setup/fans the necessary binaries.
But if we need to cross compiling the code follow the below instructions:

``` bash
sudo apt-get install git build-essential gcc-aarch64-linux-gnu binutils-aarch64-linux-gnu
wget http://www.airspayce.com/mikem/bcm2835/bcm2835-1.71.tar.gz
tar zxvf bcm2835-1.71.tar.gz
cd bcm2835-1.71
./configure --build x86_64-pc-linux-gnu --host aarch64-linux-gnu 
make

cd ..
git clone https://gist.github.com/1c13096c4cd675f38405702e89e0c536.git
cd 1c13096c4cd675f38405702e89e0c536
make CC=aarch64-linux-gnu-gcc LIBS="../bcm2835-1.71/src/libbcm2835.a -I../bcm2835-1.71/src/"
```

!!! note
    To print the current temperature of the node, we can use
    ```
    cpu=$(</sys/class/thermal/thermal_zone0/temp)
    echo "$((cpu/1000)) c"
    ```

## Generating the kube-vip manifests

```bash
curl https://kube-vip.io/manifests/rbac.yaml > kube-vip-manifest.yaml

### Append --- in the file

sudo docker run --network host \
--rm plndr/kube-vip:v0.5.0 manifest daemonset \
--interface eth0 \
--address zeus.intra \
--inCluster \
--taint \
--controlplane \
--arp \
--leaderElection | sudo tee --append kube-vip-manifest.yaml

# The default configuration always downloads the image (even if it is present). 
# To reduce the amount of data that we download from docker hub we will change it by setting the pull policy to IfNotPresent in:
```
## Resources:

* https://kube-vip.io/usage/k3s/
* https://github.com/212850a/k3s-ansible/commit/930f32432d72fd911822e6a29f253ff545a2a2ee