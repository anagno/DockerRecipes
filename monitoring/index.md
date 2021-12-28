
Without a good monitoring system, we will not be able to find the problems that our 
cluster has. So we will be using the prometheus stack to monitor the whole cluster:


```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update

kubectl create namespace monitoring

helm intrall --namespace monitoring monitoring prometheus-community/kube-prometheus-stack -f values.yaml \
    --version v26.0.0 --set grafana.adminPassword=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)
kubectl apply -f monitoring-ingress.yaml
kubectl apply -f monitoring-ingress-public.yaml
```

To get the password:

```bash
kubectl -n monitoring get secret monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 -d
```

Now we can start adding some more dashboards

* Monitoring the storage cluster:

```bash
kubectl apply -f storage/service-monitor.yaml
kubectl apply -f storage/storage-dashboard.yaml
```

* Monitoring the proxy:

```bash
kubectl apply -f proxy/traefik-dashboard-service.yaml
kubectl apply -f proxy/traefik-service-monitor.yaml
kubectl apply -f proxy/traefik-dashboard-1.yaml
kubectl apply -f proxy/traefik-dashboard-2.yaml
```

* General dashboards:

```bash
kubectl apply -f dashboards/alerts-2-dashboard.yaml
kubectl apply -f dashboards/cluster-details-dashboard.yaml
kubectl apply -f dashboards/volumes-dashboard.yaml
kubectl apply -f dashboards/node-exporter.yaml
kubectl apply -f dashboards/node-overview.yaml

```

```bash
# kubectl delete -f dashboards/cluster-metrics-dashboard.yaml
kubectl apply -f dashboards/cert-manager-dashboard.yaml
# You might need to issue a certificate to refresh the dashboard
```

## Usefull commands:

```
kubectl -n monitoring port-forward service/monitoring-kube-prometheus-prometheus 9090:9090
```

## Resources

* https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack
* https://traefik.io/blog/capture-traefik-metrics-for-apps-on-kubernetes-with-prometheus/


* https://grafana.com/grafana/dashboards/14127



Important issues:
* https://github.com/kubernetes/kube-state-metrics/pull/1237


