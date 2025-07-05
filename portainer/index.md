# Portainer

To administrate some aspects of the cluster we use portainer

``` bash
kubectl create namespace pantheon
kubectl apply -f storage.yaml


helm repo add portainer https://portainer.github.io/k8s/
helm repo update
helm install portainer portainer/portainer  -f values.yaml --namespace pantheon --version 1.0.69

kubectl apply -f ingressroute.yaml
kubectl apply -f vpa.yaml
```

To set up the nextcloud with oidc from goauthentik follow the instructions from:

* https://goauthentik.io/integrations/services/portainer/