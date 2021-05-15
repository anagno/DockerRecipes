# Cluster initialization

Once the nodes are up and running we can go ahead and initialize our cluster.
We are planning of having a high available cluster, so we will be using a couple
of more services than strictly necessary. This is done, to make certain that 
the failure of a single node will not bring down the hole cluster.

## Initialize the cluster

To have a highly available control-plane for kubernetes, we are going to use 
[kube-vip](https://kube-vip.io). Although kube-vip provides also a load-balancer,
currently we are not going to use it, because it seemed imature as a project.
But the highly available control-plane was stable.

For the first node, we will have to generate first the vip.yaml and then initialize
the cluster. 

!!! note
    But according to the 
    [documentation](https://github.com/kube-vip/kube-vip/blob/main/docs/hybrid/static/index.md#additional-nodes)
    we have first to join the cluster and afterwards generate the vip.yaml

``` bash
sudo docker run --network host \
--rm plndr/kube-vip:v0.3.4 manifest pod \
--interface eth0 \
--address STATIC_IP_THAT_IS_AVAILABLE\
--controlplane \
--arp \
--leaderElection | sudo tee /etc/kubernetes/manifests/vip.yaml
```

The default configuration always downloads the image (even if it is present).
To reduce the amount of data that we download from docker hub we will change it
by setting the pull policy to ```IfNotPresent``` in:

``` bash
sudo nano /etc/kubernetes/manifests/vip.yaml
```
!!! note
    There is a [bug](https://github.com/kube-vip/kube-vip/issues/207) in kube-vip,
    so check also that the image source is correct


Afterwards we can initialize the cluster:

``` bash
TOKEN=$(sudo kubeadm token generate)
echo $TOKEN

sudo kubeadm init --token=${TOKEN} \
--kubernetes-version=v1.21.0 \
--pod-network-cidr=10.244.0.0/16 \
--control-plane-endpoint "STATIC_IP_THAT_IS_AVAILABLE:6443" \
--upload-certs
```

We should see something like:

``` bash
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join REDACTED

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

  kubeadm join REDACTED
```

## Install a CNI add-on

Now that the first node is initialized we can also install our [CNI add-on](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/).

I went with flannel:

``` bash
kubectl apply -f https://github.com/coreos/flannel/raw/v0.14.0-rc1/Documentation/kube-flannel.yml
```

!!! note
    At the moment of the writing, because I used the latest release of kubernetes,
    I had to use the release candidate of the kube-flannel

## Joining the rest of the nodes

Now we can continue with the rest of the nodes

### Master nodes

Just follow the instructions that were printed in the previous steps.

``` bash
sudo kubeadm join REDACTED --control-plane
    
sudo docker run --network host \
--rm plndr/kube-vip:v0.3.4 manifest pod \
--interface eth0 \
--address STATIC_IP_THAT_IS_AVAILABLE \
--controlplane \
--arp \
--leaderElection | sudo tee /etc/kubernetes/manifests/vip.yaml

sudo nano /etc/kubernetes/manifests/vip.yaml
# Change the image pull policy to IfNotPresent

sudo reboot

```

### Worker nodes

Similary

``` bash
sudo kubeadm join REDACTED
```

## Troubleshooting

### Controloning the cluster from other computers

Just install ``kubectl`` to the other computer and copy the necessary files

``` bash
sudo snap install kubectl
# Copy the config file to the local computer in .kube folder
```

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