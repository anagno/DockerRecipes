# Assing limits to the pods

## Vertical Pod Autoscaler

To automate the requests and the limits of the pods we will use the 
[Vertical Pod Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler)
There is no [official](https://github.com/kubernetes/autoscaler/issues/3068) helm repo, but we can use 
an unofficial one

``` bash
kubectl create namespace scaler
helm repo add cowboysysop https://cowboysysop.github.io/charts/
helm repo update
helm install vpa cowboysysop/vertical-pod-autoscaler --namespace scaler -f values.yaml --version v9.8.1
```

!!!Note
    VPA can retrieve data also from Prometheus, but I do not want to couple 
    VPA with my monitoring stack. So for the moment I do not use prometheus 
    to retrieve data


For the reason for not defining limits, take a look at: 
* https://www.linkedin.com/pulse/kubernetes-make-your-services-faster-removing-cpu-limits-eric-khun/
* https://home.robusta.dev/blog/stop-using-cpu-limits



## Resources:

* https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler
* https://gist.github.com/sherifkayad/1b4e4df408e1be357168a38e1980b9a5
* https://github.com/cowboysysop/charts/tree/master/charts/vertical-pod-autoscaler
* https://artifacthub.io/packages/helm/cowboysysop/vertical-pod-autoscaler
* https://povilasv.me/vertical-pod-autoscaling-the-definitive-guide/
* https://goldilocks.docs.fairwinds.com/
* https://www.youtube.com/watch?v=UE7QX98-kO0
* https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/#create-a-limitrange-and-a-pod