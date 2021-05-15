# Sharing secrets accross namespaces

## Kubed

Kubernets does not share secrets accross namespaces. But some times it is 
usefull to be able to share secrets accross namespaces. For that purpose 
we [kubed service](https://appscode.com/products/kubed/v0.12.0/guides/config-syncer/intra-cluster/)


``` bash
helm repo add appscode https://charts.appscode.com/stable/
helm repo update
helm install kubed appscode/kubed -f kubed_values.yaml --version v0.12.0 --namespace kube-system
```

The `kubed_values.yaml` I used is:

``` yaml
# https://github.com/kubeops-tools/kubed/blob/v0.12.0/charts/kubed/values.yaml

# Number of Kubed operator replicas to create (only 1 is supported)
replicaCount: 1

operator:
  registry: appscode
  repository: kubed
  tag: v0.12.0
  # Compute Resources required by the operator container
  resources: {}
  securityContext: {}

# Container image pull policy
imagePullPolicy: IfNotPresent

serviceAccount:
  create: true

apiserver:
  # Port used by Kubed server
  securePort: "8443"
  healthcheck:
    # healthcheck configures the readiness and liveliness probes for the operator pod.
    enabled: false
  servingCerts:
    generate: true

enableAnalytics: false

config:
  clusterName: cluster
```

After the service is deployed we can create secrets in any namespace that can be synced
accross namespaces:

``` bash
kubectl -n general create secret generic no-reply-mail --from-literal=password=PASSWORD
kubectl -n general annotate secret no-reply-mail kubed.appscode.com/sync=""
```

## Resources
* https://appscode.com/products/kubed/v0.12.0/guides/config-syncer/intra-cluster/