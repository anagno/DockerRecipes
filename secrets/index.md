# Sharing secrets accross namespaces

## Kubernetes replicator

Kubernets does not share secrets accross namespaces. But some times it is 
usefull to be able to share secrets accross namespaces. For that purpose 
we use the [kubernetes-replicator](https://github.com/mittwald/kubernetes-replicator)


``` bash
helm repo add mittwald https://helm.mittwald.de
helm repo update
helm install kubernetes-replicator mittwald/kubernetes-replicator -f values.yaml --version v2.10.2 --namespace kube-system
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
kubectl -n general annotate secret no-reply-mail replicator.v1.mittwald.de/replicate-to="*"
```

## Resources
* https://appscode.com/products/kubed/v0.12.0/guides/config-syncer/intra-cluster/