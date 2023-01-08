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
kubectl run --rm --tty --stdin --image quay.io/coreos/etcd:v3.5.4 etcdctl --overrides='{"apiVersion":"v1","kind":"Pod","spec":{"hostNetwork":true,"restartPolicy":"Never","securityContext":{"runAsUser":0,"runAsGroup":0},"containers":[{"command":["/bin/sh"],"image":"docker.io/rancher/coreos-etcd:v3.5.4-arm64","name":"etcdctl","stdin":true,"stdinOnce":true,"tty":true,"volumeMounts":[{"mountPath":"/var/lib/rancher","name":"var-lib-rancher"}]}],"volumes":[{"name":"var-lib-rancher","hostPath":{"path":"/var/lib/rancher","type":"Directory"}}],"nodeSelector":{"node-role.kubernetes.io/etcd":"true"}}}'

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


## Defrag the etcd nodes 

```sh
ansible erato -b -m ansible.builtin.shell -a 'curl -L https://github.com/etcd-io/etcd/releases/download/v3.5.4/etcd-v3.5.4-linux-arm64.tar.gz -o /tmp/etcd-v3.5.4-linux-arm64.tar.gz'
ansible erato -b -m ansible.builtin.shell -a 'tar xzvf /tmp/etcd-v3.5.4-linux-arm64.tar.gz -C /tmp --strip-components=1'
ansible erato -b -m ansible.builtin.shell -a '/tmp/etcdctl version' 
ansible erato -b -m ansible.builtin.shell -a '/tmp/etcdctl --key /var/lib/rancher/k3s/server/tls/etcd/client.key --cert /var/lib/rancher/k3s/server/tls/etcd/client.crt --cacert /var/lib/rancher/k3s/server/tls/etcd/server-ca.crt member list' 
ansible erato -b -m ansible.builtin.shell -a '/tmp/etcdctl --key /var/lib/rancher/k3s/server/tls/etcd/client.key --cert /var/lib/rancher/k3s/server/tls/etcd/client.crt --cacert /var/lib/rancher/k3s/server/tls/etcd/server-ca.crt defrag --endpoints=[192.168.179.101:2379] ' 
ansible erato -b -m ansible.builtin.shell -a '/tmp/etcdctl --key /var/lib/rancher/k3s/server/tls/etcd/client.key --cert /var/lib/rancher/k3s/server/tls/etcd/client.crt --cacert /var/lib/rancher/k3s/server/tls/etcd/server-ca.crt defrag --cluster' 
```

Resources:
* https://gist.github.com/superseb/0c06164eef5a097c66e810fe91a9d408

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

## Change the USB_MSD_PWR_OFF_TIME

Some times the USB devices are not [re-attache](https://github.com/raspberrypi/rpi-eeprom/issues/361)
after a reboot. 

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

sudo rpi-eeprom-config --edit

# Add USB_MSD_PWR_OFF_TIME=0 in the end
sudo reboot
```

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
--rm plndr/kube-vip:v0.5.7 manifest daemonset \
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

To update it in the nodes `ansible masters -b -m copy -a "src=cluster_setup/kube-vip-manifest.yaml dest=/var/lib/rancher/k3s/server/manifests/vip.yaml"`

## Drain a node

```bash
kubectl drain <node name>
ansible <node name> -b -m community.general.shutdown
kubectl uncordon <node name>
```

## Security benchmark 

We can use the:

* [kube-bench](https://github.com/aquasecurity/kube-bench)

    ```sh
    curl https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job.yaml > kube-bench-job.yaml
    kubectl apply -f kube-bench-job.yaml
    kubectl logs kube-bench-kp8sv
    kubectl delete pod kube-bench-kp8sv
    kubectl delete -f kube-bench-job.yaml
    rm kube-bench-job.yaml
    ```

## Resources:

* https://kube-vip.io/usage/k3s/
* https://github.com/212850a/k3s-ansible/commit/930f32432d72fd911822e6a29f253ff545a2a2ee
* https://github.com/k3s-io/k3s/issues/2732#issuecomment-749484037
* https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/



## Go over them

https://hub.docker.com/r/osminogin/tor-simple
https://tufin.medium.com/how-to-use-a-proxy-with-go-http-client-cfc485e9f342

https://ikarus.sg/using-zram-to-get-more-out-of-your-raspberry-pi/

## Troubleshooting

### Updating the configuration

For updating the configuration of the running cluster we need acces to one of the master nodes
(because we need access to the kubadm tool). So log in to the master node and:

``` bash
kubectl get cm -o yaml -n kube-system kubeadm-config > kubeadm-config.yaml
```

Now we can edit our configuration. To validate them we can run 

``` bash
kubeadm upgrade diff --config kubeadm-config.yaml
```

and to apply them just execute: 

```
kubeadm upgrade apply --config kubeadm-config.yaml
```


Resources:

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


In the case a master node fails we will have to remove him from the etcd cluster

Resources:
 * https://github.com/kubernetes/kubeadm/issues/1300#issuecomment-464955084

```
kubectl delete node <name>

kubectl -n kube-system exec -it etcd-melpomene -- etcdctl --endpoints https://127.0.0.1:2379 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key member list

kubectl -n kube-system exec -it etcd-melpomene -- etcdctl --endpoints https://127.0.0.1:2379 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key member remove 4985985992cc00d

# Remove from kubeadm-config

kubectl -n kube-system get cm kubeadm-config -o yaml > /tmp/conf.yml
manually edit /tmp/conf.yml to remove the old server
kubectl -n kube-system apply -f /tmp/conf.yml

```

```
sudo apt-get install -y --allow-change-held-packages kubeadm=1.21.x-00 kubelet=1.21.x-00 kubectl=1.21.x-00

sudo apt-get install -y --allow-change-held-packages kubeadm=1.21.2-00 kubelet=1.21.2-00 kubectl=1.21.2-00
sudo kubeadm upgrade apply v1.21.2
sudo apt-mark hold kubelet kubeadm kubectl
```

Take a look at:

* https://github.com/nextcloud/helm
* https://geek-cookbook.funkypenguin.co.nz/recipes/jellyfin/
* https://jellyfin.org/docs/general/server/users/index.html
* https://github.com/home-cluster/jellyfin-openshift/blob/master/deploy/deployment.yaml
* https://github.com/home-cluster/jellyfin-openshift/blob/master/deploy/deployment.yaml
* https://github.com/cdwv/bitwarden-k8s/blob/master/chart/bitwarden-k8s/templates/deployment.yaml
* https://github.com/guerzon/bitwarden-kubernetes
* https://greg.jeanmart.me/2020/04/13/self-host-your-password-manager-with-bitward/

### Removing a master node

If a master node fails, we will have also to update the etcd nodes, to remove it
from the etcd cluster

``` bash
kubectl delete node <name>

kubectl -n kube-system exec -it etcd-melpomene -- etcdctl --endpoints https://127.0.0.1:2379 \
    --cacert /etc/kubernetes/pki/etcd/ca.crt \
    --cert /etc/kubernetes/pki/etcd/server.crt \
    --key /etc/kubernetes/pki/etcd/server.key member list

kubectl -n kube-system exec -it etcd-melpomene -- etcdctl --endpoints https://127.0.0.1:2379 \
    --cacert /etc/kubernetes/pki/etcd/ca.crt \
    --cert /etc/kubernetes/pki/etcd/server.crt \
    --key /etc/kubernetes/pki/etcd/server.key member remove <node_id>

# Remove from kubeadm-config

kubectl -n kube-system get cm kubeadm-config -o yaml > /tmp/conf.yml
manually edit /tmp/conf.yml to remove the old server
kubectl -n kube-system apply -f /tmp/conf.yml
```

### General problems with limits

I think these issues:

- https://github.com/kubernetes/kubernetes/pull/75682
- https://github.com/kubernetes/kubernetes/issues/51135
- https://github.com/dotnet/aspnetcore/issues/29150

are still valid because I saw thottling when I monitored the cluster. 
So I deactivated the limits for the CPU. Theoretically it should not cause problems, 
only some throttling if the cpu is overused in the node.

## Resources
* https://github.com/kubernetes/kubeadm/issues/1300#issuecomment-464955084
* https://blog.honosoft.com/2020/01/31/kubeadm-how-to-upgrade-update-your-configuration/
* https://github.com/kubernetes/kubeadm/issues/1464


    #- name: Check packages
    #  package_facts:
    #    manager: auto

    #- name: Install open-iscsi
    #  package:
    #    name: open-iscsi
    #  when: not "'open-iscsi' in ansible_facts.packages"


    #- name: check services
    #  service_facts:

    #- name: print
    #  debug:
    #    msg: "iscsid is {{ ansible_facts.services['iscsid.service']['status'] }}"
    #  when: "'iscsid.service' in ansible_facts.services"


ansible masters -m ansible.builtin.shell -a 'cat /boot/firmware/cmdline.txt'
ansible masters -b -m ansible.builtin.shell -a 'cat /var/lib/rancher/k3s/server/manifests/coredns-local.yaml'
ansible masters -b -m copy -a "src=cluster_setup/coredns.yaml dest=/var/lib/rancher/k3s/server/manifests/coredns-local.yaml" 


ansible masters -b -m ansible.builtin.shell -a 'rm /var/lib/rancher/k3s/server/manifests/coredns.yaml'
ansible erato -b -m ansible.builtin.shell -a 'ls -al /var/lib/rancher/k3s/server/manifests/'

ansible erato -b -m ansible.builtin.service -a 'name=k3s state=restarted'


ansible erato -b -m ansible.builtin.shell -a 'k3s etcd -h'