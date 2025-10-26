# Cluster Initialization

Now that are nodes are up and running we can start with the 
initialization of the  k3s cluster.
Before the initializatio l,n reserve a domain name in your router
and reserve IPs for the Virtual IP of the control plnae.
The scripts we have in ansible makes it easy. Just execute:

```bash
ansible-playbook cluster_setup/setup_cluster.yml
```

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
    Avoid the use of documented taint k3s-controlplane=true:NoExecute used to avoid deployment of pods on master node.
    We are interested on running certain pods on master node, like the ones needed to collect logs/metrics from the master node.
    Instead, use the taint `node-role.kubernetes.io/control-plane:NoSchedule`. K3S common services: core-dns, metric-service, 
    service-lb are configured with tolerance to node-role.kubernetes.io/control-plane taint, so they will be scheduled on master node.
    So use `kubectl taint nodes myserver node-role.kubernetes.io/control-plane:NoSchedule`

## Resources

* https://rancher.com/docs/k3s/latest/en/cluster-access/#accessing-the-cluster-from-outside-with-kubectl
* https://rpi4cluster.com/k3s/k3s-kube-setting/
* https://github.com/k3s-io/k3s
* https://github.com/robipozzi/windfire-raspberry
* https://github.com/geerlingguy/raspberry-pi-dramble
* https://ikarus.sg/kubernetes-with-k3s/
* https://gist.github.com/LarsNieuwenhuizen/03c224e50871e123e4376f0518083cb1
* https://picluster.ricsanfre.com