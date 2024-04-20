# Load balancer

## Metallb

For the load balancer we are using the [metallb](https://metallb.universe.tf/).
Theoretically we could have used the integratetd one from the k3s, but the 
integrated one is not able to handle virtual IPs. 
Alternative we could have used the kube-vip but the provided controller from
kube-vip is not mature enough. If we ever decide to use kube-vip then we will have 
to update the kube-vip daemon set and activate the `--service` parameter.

Metallb provides a helm chart. So the installation is quite simple:

``` bash
helm repo add metallb https://metallb.github.io/metallb
kubectl create namespace load-balancer
helm install --namespace load-balancer load-balancer metallb/metallb -f values.yaml --version 0.14.5
# Wait for the full deployment of the services
kubectl apply -f IPAddressPool.yaml
kubectl apply -f vpa.yaml
```

The `values.yaml` I used is:

```
--8<-- "load_balancer/values.yaml"
```

!!!Note
    Initially the prometheus monitoring should not be activate until we deploy the 
    monitoring stack. Afterwards we can activate it.

## Validation
To make sure that the assignment of the IPs is working,
we will create a small testing service 

``` bash
kubectl create namespace kube-verify
kubectl apply -f verify.yaml
```

With `verify.yaml`:

``` yaml
--8<-- "load_balancer/verify.yaml"
```

The test service should be now availalbe in the IP we specified in verify.
Once we make certain that everything is working we can deleting the validation service:

``` bash
kubectl delete -f verify.yaml
kubectl delete namespace kube-verify
```

## Userfull commands:

To get all the assigned IPs we just execute:

```bash
kubectl get services -o wide --all-namespaces | grep --color=never -E 'LoadBalancer|NAMESPACE'
```

## Resources:

* https://opensource.com/article/20/7/homelab-metallb
* https://www.devtech101.com/2019/02/23/using-metallb-and-traefik-load-balancing-for-your-bare-metal-kubernetes-cluster-part-1/
* https://metallb.universe.tf/configuration/
* https://github.com/metallb/metallb/issues/308
* https://opensource.com/article/20/7/homelab-metallb


https://github.com/inlets/inlets-operator


IPV6

https://kubernetes-sigs.github.io/external-dns/v0.14.0/sources/service/#clusterip-headless
https://kubernetes-sigs.github.io/external-dns/v0.14.0/tutorials/traefik-proxy/#manifest-for-clusters-with-rbac-enabled
https://kubernetes-sigs.github.io/external-dns/v0.14.0/tutorials/hostport/
https://metallb.universe.tf/troubleshooting/
kubectl get nodes -o jsonpath='{.items[*].spec.podCIDR}'

https://docs.k3s.io/cli/server#networking 


 --cluster-cidr=10.42.0.0/16,fd42::/56 --service-cidr=10.43.0.0/16,fd43::/112

https://github.com/kubernetes/kubernetes/issues/81677#issuecomment-524351347

https://github.com/kubernetes/kubernetes/issues/111671


https://github.com/k3s-io/docs/pull/103/files
