## DNS resolution



By default kubernetes uses it is internal dns, which is only aware of the internal services.
During the deployement of a pod we can set the `dnsPolicy`. In my network, I have my internal
DNS server, which I prefer to be used after the internal resolution of the services. So I will
update the coredns service to use my nameserver for resolving the external address. For that 
purpose I will edit the current forwarders

!!!Note
    Signs you need to do this step, is that the containers can not resolve external
    domain names. For example I had that problem with authelia and the setup of the smtp

!!!Note
    Unfortunatelly k3s does not provide a way to customize the coredns configurations

What I am mentioning above is just a work around. To adapt the coredns, just execute:

```bash
ansible-playbook networking/coredns.yml
kubectl -n kube-system rollout restart deployment coredns
```


I have tried in multiple ways to work around the limitation that k3s is not 
providing a way of customizing coredns. I have tried to:

* to create a custom d_custom_coredns.yml file in the manifests folder
* to specify the --resolv-conf during the initialization of the k3s
* and even to specify the ndots in the containers.

In generall I could not make it work, and from time to time I will have to re-execute
the coredns.yml playbook to fix the setting in the coredns.



# Usefull Commands

```bash
KUBE_EDITOR="nano" kubectl -n kube-system edit configmaps coredns -o yaml
```


# Resources
* https://www.danielstechblog.io/setting-custom-upstream-nameservers-for-coredns-in-azure-kubernetes-service/
* https://github.com/coredns/coredns/issues/2087
* https://github.com/k3s-io/k3s/issues/4087#issuecomment-929374460
* https://rancher.com/docs/k3s/latest/en/installation/install-options/agent-config/
* https://github.com/k3s-io/k3s/pull/4397
* https://github.com/k3s-io/k3s/issues/1797
* https://github.com/k3s-io/k3s/issues/53
* https://pracucci.com/kubernetes-dns-resolution-ndots-options-and-why-it-may-affect-application-performances.html