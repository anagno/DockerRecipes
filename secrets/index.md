# Sharing secrets accross namespaces

## Config Syncer

Kubernets does not share secrets accross namespaces. But some times it is 
usefull to be able to share secrets accross namespaces. For that purpose 
we use the [Config Syncer service](https://appscode.com/products/kubed/v0.12.0/guides/config-syncer/intra-cluster/)

``` bash
helm repo add appscode https://charts.appscode.com/stable/
helm repo update
helm install kubed appscode/kubed -f values.yaml --version v0.13.2 --namespace kube-system
kubectl apply -f vpa.yaml
```

The `values.yaml` I used is:


``` yaml
--8<-- "secrets/values.yaml"
```


After the service is deployed we can create secrets in any namespace that can be synced
accross namespaces:

``` bash
kubectl -n general create secret generic no-reply-mail --from-literal=password=PASSWORD
kubectl -n general annotate secret no-reply-mail kubed.appscode.com/sync=""
```

## Resources
* https://appscode.com/products/kubed/v0.12.0/guides/config-syncer/intra-cluster/