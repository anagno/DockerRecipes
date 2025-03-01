
Without a good monitoring system, we will not be able to find the problems that our 
cluster has. So we will be using the prometheus stack to monitor the whole cluster:


```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update

kubectl create namespace monitoring


# Follow the instructions for goauthentik
# https://goauthentik.io/integrations/services/grafana/

kubectl create secret generic authentik-secret --namespace monitoring \
  --from-literal=client_id=ID_FROM_AUTHENTIK \
  --from-literal=client_secret=SECRET_FROM_AUTHENTIK

helm install --namespace monitoring monitoring prometheus-community/kube-prometheus-stack -f values.yaml \
    --version v69.6.0 --set grafana.adminPassword=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)
kubectl apply -f monitoring-ingress-public.yaml
kubectl apply -f vpa.yaml


# https://github.com/grafana/helm-charts/issues/3300
helm install --namespace monitoring loki grafana/loki-stack -f loki-values.yaml --version 2.10.2

helm install --namespace monitoring event-explorter bitnami/kubernetes-event-exporter -f event-exporter-values.yaml --version 2.9.3

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
kubectl apply -f proxy/traefik-dashboard.yaml
kubectl apply -f proxy/traefik-dashboard-loki.yaml
```

* General dashboards:

```bash
kubectl apply -f dashboards/alerts-summary-dashboard.yaml
kubectl apply -f dashboards/alerts-dashboard.yaml
kubectl apply -f dashboards/cluster-details-dashboard.yaml
kubectl apply -f dashboards/cluster-details-namespaces.yaml
kubectl apply -f dashboards/cluster-details-nodes.yaml
kubectl apply -f dashboards/volumes-dashboard.yaml
kubectl apply -f dashboards/node-exporter.yaml
kubectl apply -f dashboards/node-overview.yaml
kubectl apply -f dashboards/vpa-dashboard.yaml
kubectl apply -f dashboards/event-exporter.yaml
kubectl apply -f dashboards/loki-search.yaml
```

* Load-balancer dashboard (if it has been activated in the helm chart): 

```bash
kubectl apply -f dashboards/metallb-dashboard.yaml
```

* Certificates dashboard (if it has been activated in the helm chart): 

```bash
kubectl apply -f dashboards/cert-manager-dashboard.yaml
```

* External-dns dashboard (if it has been activated in the helm chart): 

```bash
kubectl apply -f dashboards/externa-dns-dashboard.yml
```

* Authorization dashboard (if it has been activated in the helm chart): 

```bash
kubectl apply -f dashboards/authentik-dashboard.yaml
```

```bash
helm repo add kuberhealthy https://kuberhealthy.github.io/kuberhealthy/helm-repos
helm install -n monitoring kuberhealthy kuberhealthy/kuberhealthy -f kuberhealthy-values.yaml --version 105
```

## Usefull commands:

```
kubectl -n monitoring port-forward service/monitoring-kube-prometheus-prometheus 9090:9090
```

TODO add as dashboard 
https://grafana.com/grafana/dashboards/15760-kubernetes-views-pods/
https://grafana.com/grafana/dashboards/21410-kubernetes-overview/


## Resources

* https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack
* https://traefik.io/blog/capture-traefik-metrics-for-apps-on-kubernetes-with-prometheus/


* https://grafana.com/grafana/dashboards/15398
* https://grafana.com/grafana/dashboards/15761



Important issues:
* https://github.com/kubernetes/kube-state-metrics/pull/1237



https://github.com/kubernetes/kube-state-metrics/issues/2041
https://github.com/prometheus-community/helm-charts/blob/c8d79dbef2b93fad69e458da2ecd09920fb786a7/charts/kube-state-metrics/values.yaml#L338


