
Without a good monitoring system, we will not be able to find the problems that our 
cluster has. So we will be using the prometheus stack to monitor the whole cluster:


```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update


ansible-playbook monitoring/firewall.yml

kubectl create namespace monitoring


# Follow the instructions for goauthentik
# https://goauthentik.io/integrations/services/grafana/

kubectl create secret generic authentik-secret --namespace monitoring \
  --from-literal=client_id=ID_FROM_AUTHENTIK \
  --from-literal=client_secret=SECRET_FROM_AUTHENTIK

helm upgrade --namespace monitoring monitoring prometheus-community/kube-prometheus-stack -f values.yaml \
    --version v84.5.0 --set grafana.adminPassword=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)
kubectl apply -f monitoring-ingress-public.yaml
kubectl apply -f vpa.yaml


# https://medium.com/@rayanee/building-a-complete-monitoring-stack-on-kubernetes-with-prometheus-loki-and-grafana-32d6cc1a45e0
# https://weepyadmin.com/docs/devops/install_prometheus_loki_in_k8s/
# https://grafana.com/grafana/dashboards/16976-kubernetes-loki-logs/
# https://hodovi.cc/blog/kubernetes-events-monitoring-with-loki-alloy-and-grafana/

# https://jay75chauhan.medium.com/kubernetes-observability-metrics-logs-and-traces-with-grafana-stack-d57882dbe639
# https://github.com/jay75chauhan/k8s-Observability/blob/main/values.yaml
helm repo add grafana-community https://grafana-community.github.io/helm-charts
helm upgrade --namespace monitoring loki grafana-community/loki -f loki_values.yaml --version 13.2.4

helm repo add grafana https://grafana.github.io/helm-charts
helm upgrade --namespace monitoring alloy grafana/alloy -f alloy_values.yaml --version 1.8.0
# TODO find an alternative for loki and alloy
# Take a look at https://github.com/grafana/k8s-monitoring-helm/blob/main/charts/k8s-monitoring/values.yaml
# https://medium.com/@rayanee/building-a-complete-monitoring-stack-on-kubernetes-with-prometheus-loki-and-grafana-32d6cc1a45e0


# https://grafana.com/grafana/dashboards/24593-traefik-opentelemetry/
# https://grafana.com/grafana/dashboards/16976-kubernetes-loki-logs/
# https://grafana.com/grafana/dashboards/23100-kubernetes-events-overview/
# https://grafana.com/grafana/dashboards/23101-kubernetes-events-timeline/
# https://grafana.com/grafana/dashboards/17501-traefik-via-loki/
# https://grafana.com/grafana/dashboards/24593-traefik-opentelemetry/

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
```

* General dashboards:

```bash
kubectl apply -f dashboards/alerts-summary-dashboard.yaml
kubectl apply -f dashboards/alerts-dashboard.yaml
kubectl apply -f dashboards/volumes-dashboard.yaml
kubectl apply -f dashboards/node-exporter.yaml
kubectl apply -f dashboards/hpa-dashboard.yaml
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


