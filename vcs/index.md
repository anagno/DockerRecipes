# Gitea

To store source code we are going to use gitea

```bash
kubectl create namespace vcs

kubectl create secret generic gitea --namespace vcs \
  --from-literal=username=caretaker \
  --from-literal=password=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64) 

kubectl create secret generic gitea-database --namespace vcs \
  --from-literal=postgres-password=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64) \
  --from-literal=password=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)

# Follow first the steps here and use the strings from there
kubectl create secret generic gitea-oauth --namespace vcs \
  --from-literal=key= ... \
  --from-literal= ...

kubectl apply -f storage.yaml

helm repo add gitea-charts https://dl.gitea.com/charts/
helm repo update
helm install vcs gitea-charts/gitea -f values.yaml --namespace vcs --version 10.3.0

kubectl apply -f ingressroute.yaml


```
## Resources: 

* https://gitea.com/gitea/helm-chart/issues/459