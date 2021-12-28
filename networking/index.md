## DNS resolution

!!!Note
    On updates it resets (e.g updating the k3s). FIX IT!!!
    https://github.com/k3s-io/k3s/issues/1797

By default kubernetes uses it is internal dns, which is only aware of the internal services.
During the deployement of a pod we can set the `dnsPolicy`. In my network, I have my internal
DNS server, which I prefer to be used after the internal resolution of the services. So I will
update the coredns service to use my nameserver for resolving the external address. For that 
purpose I will edit the current forwarders

```bash
KUBE_EDITOR="nano" kubectl -n kube-system edit configmaps coredns -o yaml
```

The line we want to change is `forward . /etc/resolv.conf` to
`forward . 192.168.179.2 208.67.222.222 208.67.220.220 `

After we do that we will have to restart the service:

``` bash
kubectl -n kube-system rollout restart deployment coredns
kubectl -n kube-system logs -f -l "k8s-app=kube-dns"
```
!!!Note
    Signs you need to do this step, is that the containers can not resolve external
    domain names. For example I had that problem with authelia and the setup of the smtp


# Resources
* https://www.danielstechblog.io/setting-custom-upstream-nameservers-for-coredns-in-azure-kubernetes-service/

