# Storage

Stateless applications/services are fun, but eventually we would 
like to store some state/data. The options we have are:

* **Rook/Ceph:** I have tested rook and the options are and features 
are really extensive and very good. It is quite stable, but we are
facing the limitations of our platform. Raspberry Pi is an arm64 platform.
Ceph does support arm64, but some other usefull features (e.g. running NFS 
endpoints) did not have yet arm64 images. Furthermore it needs a lot of 
CPU and memory. To deploy a fully functional ceph cluster we need at 
least 10 Raspberry Pi (and they have to be raspberry pi 4 with at 
least 4GB of RAM). So technically it is possible and I have done it, but 
the headaches are too many. Do not get me wrong. I am pushing what is 
feasible by deploying it to Raspberry Pies and it still worked. So kudos to
Ceph. But probably it is an overcomplicated solution for what I want to do. 
* **NFS:** We can use an external nfs storage server. Probably this one will 
work like charm, but it is a single point of failure. 
* **Longhorn:** This is another native Kubernetes storage, which seems quite 
promising. They only disadvantage is that it does not support 
[sharding](https://github.com/longhorn/longhorn/issues/1061). So we will be 
limiting our selfes to the maximum size of our hard drives in the cluster.
For my current use cases, it is more that fine. Furthermore it is supported 
from k3s. So we will go with it and see what happens.


## Preparing the nodes

Before we are able to install longhorn, we have to prepare the nodes 
and mount the disks. To be able to do that we will have to able to find
out where the devices are located. So we execute:

```bash
ansible disks -b -m ansible.builtin.shell -a 'blkid'
```

We will get something like:

```bash 
/dev/sda: UUID="blah-blah-...-blah" TYPE="LVM2_member"
```

So complete these details in our hosts file in the section of the disks. E.g.:

```yaml
    disks:
      hosts:
        node_1:
          disk_path: /dev/sda
```

Then we can execute our playbook

```bash
ansible-galaxy collection install ansible.posix
ansible-galaxy collection install community.general
ansible-playbook storage/setup_storage.yml
```

!!!Warning
    I have no idea why, but if I connect the Sata to USB3 cable I have
    to the USB3 ports, after a reboot the drive will not re-connect.
    To reconnect I have to power completely the raspberry pi (unplug
    and replug the PoE cable). Needs investigation.

!!!Note
    Be carefull to use the correct partition

!!!Note
    I used to have ceph installed on the disks and to clean I have first to execute:

    ```
    ansible disks -b -m ansible.builtin.shell -a 'sgdisk --zap-all /dev/sdb'
    ansible disks -b -m ansible.builtin.shell -a 'sudo dd if=/dev/zero of=/dev/sdb bs=1M count=100 oflag=direct,dsync'
    ansible disks -b -m ansible.builtin.shell -a 'ls /dev/mapper/ceph-* | xargs -I% -- sudo dmsetup remove %'
    ansible disks -b -m ansible.builtin.shell -a 'sudo rm -rf /dev/ceph-*'
    ```

## Installing longhorn

```bash
# We have to use the longhrn-system namespace. It is mentioned in the
# documentation of the helm chart
helm repo add longhorn https://charts.longhorn.io
helm repo update
kubectl create namespace longhorn-system
helm install longhorn longhorn/longhorn --namespace longhorn-system -f values.yaml --version 1.6.1

# The vpa is causing instability in the longhorn. So do not activate it for the moment
#kubectl apply -f vpa.yml
kubectl apply -f dashboard.yml

# Apply our storage classes 

kubectl create secret generic longhorn-crypto --namespace longhorn-system \
  --from-literal=CRYPTO_KEY_VALUE=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64) \
  --from-literal=CRYPTO_KEY_PROVIDER=secret

kubectl apply -f RepliccatedStorage.yaml
kubectl apply -f UnrepliccatedStorage.yaml
```

We have to add the disks from the UI of longhorn

* https://rpi4cluster.com/k3s/k3s-storage-setting/#add-storage
* https://longhorn.io/docs/1.2.3/volumes-and-nodes/multidisk/#add-a-disk

On how to use them take a look at https://longhorn.io/docs/1.2.2/references/examples/#block-volume

Maybe of interest:

* https://longhorn.io/docs/1.2.2/monitoring/prometheus-and-grafana-setup/#install-longhorn-servicemonitor
* https://github.com/longhorn/longhorn/issues/1859#issuecomment-907057960 for when expanding a disk

## Usefull commands 

```
kubectl -n longhorn-system logs -f -l "app=longhorn-manager" --max-log-requests 10
```

## Resources

* https://bryanbende.com/development/2021/05/15/k3s-raspberry-pi-volumes-storage
* https://github.com/gdha/pi4-longhorn
* https://gdha.github.io/pi-stories/pi-stories9/
* https://www.jericdy.com/blog/installing-k3s-with-longhorn-and-usb-storage-on-raspberry-pi
* https://longhorn.io/docs/1.2.2/advanced-resources/volume-encryption/


Take a look at https://longhorn.io/kb/troubleshooting-volume-with-multipath/
I had to do it in homados and aretusa, and I have to examine if it has to be done to all nodes ...


```sh
# https://github.com/longhorn/longhorn/issues/1826#issuecomment-1200005051
kubectl get snapshots.longhorn.io -n longhorn-system -l longhornvolume=pvc-1c16f507-ff8d-4b8b-aed4-7b108214618c | awk '/library-books-/{print $1}' | xargs kubectl -n longhorn-system delete snapshots.longhorn.io

qemu-img convert -f raw e5ec5a95 -O vmdk torrents-settings.img
sudo mount -o loop torrents-settings mount/

# https://edoceo.com/sys/qemu
modprobe nbd
qemu-nbd --connect=/dev/nbd0 disk.qcow2

qemu-nbd -d /dev/nbd0
```