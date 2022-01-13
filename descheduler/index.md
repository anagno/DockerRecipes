# Re-Balancing the pods

## Descheduler

To keep the load on the nodes balanced we will use the
[descheduler](https://github.com/kubernetes-sigs/descheduler)

``` bash
helm repo add descheduler https://kubernetes-sigs.github.io/descheduler/
helm repo update
helm install descheduler descheduler/descheduler -f values.yaml --namespace kube-system --version v0.22.1
```

## Usefull commands:

```bash
kubectl -n kube-system logs -f -l "app.kubernetes.io/instance=descheduler"
```


## Resources:

* https://github.com/kubernetes-sigs/descheduler/blob/master/docs/user-guide.md