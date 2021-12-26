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
helm install --namespace load-balancer load-balancer metallb/metallb -f values.yaml --version 0.11.0
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