# Cluster Initialization

Now that are nodes are up and running we can start with the 
initialization of the  k3s cluster.
The scripts we have in ansible makes it easy. Just execute:

```bash
ansible-playbook cluster_setup/setup_cluster.yml
```

!!!Note
    Flannel default network is not enrcypted. So the communication between the pods is unencrypted. 
    There is an experimental backend that supports Wireguard (https://github.com/flannel-io/flannel/blob/master/Documentation/backends.md#wireguard).
    I also tried with the experimental IPSec backend, but it did not work

Once the cluster is initialized we can start controlling it using kubectl.
But for that we have to install it to our local system

```bash
sudo snap install kubectl
```

We also have to copy /etc/rancher/k3s/k3s.yaml on your machine 
located outside the cluster as ~/.kube/config. 

```bash
ansible "erato" -b -m ansible.builtin.fetch -a 'src=/etc/rancher/k3s/k3s.yaml dest=~/.kube/config flat=yes'
```

Then replace “localhost” with the IP or name of your K3s server. kubectl can now manage your K3s cluster.

!!!Note
    Unlike k8s, in k3s the master nodes are elible to run containers destined for workers as it does not 
    have the `node-role.kubernetes.io/master=true:NoSchedule`. To re-introduce it (if we see that the 
    master nodes are being influenced) we can execute 
    `kubectl taint nodes myserver node-role.kubernetes.io/master=true:NoSchedule`
    We can see the taints with `kubectl get nodes -o json | jq '.items[].spec.taints'`

## Resources

* https://rancher.com/docs/k3s/latest/en/cluster-access/#accessing-the-cluster-from-outside-with-kubectl
* https://rpi4cluster.com/k3s/k3s-kube-setting/
* https://github.com/k3s-io/k3s
* https://github.com/robipozzi/windfire-raspberry
* https://github.com/geerlingguy/raspberry-pi-dramble
* https://ikarus.sg/kubernetes-with-k3s/
* https://gist.github.com/LarsNieuwenhuizen/03c224e50871e123e4376f0518083cb1