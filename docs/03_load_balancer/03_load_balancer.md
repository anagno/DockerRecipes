# Load balancer

## Metallb

For the load balancer we are using the [metallb](https://metallb.universe.tf/).
Theoretically we could have used the kube-vip but the provided controller from
kube-vip is not mature enough. If we ever decide to use kube-vip then we will have 
to update the kube-vip instance and activate the `--service` parameter.

Metallb provides a helm chart. So the installation is quite simple:

``` bash
helm repo add bitnami https://charts.bitnami.com/bitnami
kubectl create namespace load-balancer
helm install --namespace load-balancer load-balancer bitnami/metallb -f values.yaml --version 2.3.5
```

The `values.yaml` I used is:

```
--8<-- "docs/03_load_balancer/values.yaml"
```

## Validation
To make sure that the assignment of the IPs is working,
we will create a small testing service 

``` bash
kubectl create namespace kube-verify
kubectl apply -f verify.yaml
```

With `verify.yaml`:

``` yaml
--8<-- "docs/03_load_balancer/verify.yaml"
```

The test service should be now availalbe in the IP we specified in verify.
Once we make certain that everything is working we can deleting the validation service:

``` bash
kubectl delete -f verify.yaml
kubectl delete namespace kube-verify
```

## Resources
* https://opensource.com/article/20/7/homelab-metallb
* https://www.devtech101.com/2019/02/23/using-metallb-and-traefik-load-balancing-for-your-bare-metal-kubernetes-cluster-part-1/
* https://metallb.universe.tf/configuration/
* https://github.com/metallb/metallb/issues/308
* https://opensource.com/article/20/7/homelab-metallb