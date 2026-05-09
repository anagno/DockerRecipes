
Without a good monitoring system, we will not be able to find the problems that our 
cluster has. So we will be using the Victoria metrics to monitor the whole cluster:


```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add vm https://victoriametrics.github.io/helm-charts/
helm repo update

ansible-playbook monitoring/firewall.yml

kubectl create namespace monitoring


# Follow the instructions for goauthentik
# https://goauthentik.io/integrations/services/grafana/

kubectl create secret generic authentik-secret --namespace monitoring \
  --from-literal=client_id=ID_FROM_AUTHENTIK \
  --from-literal=client_secret=SECRET_FROM_AUTHENTIK


helm install --namespace monitoring prometheus-crds oci://ghcr.io/prometheus-community/charts/prometheus-operator-crds --version 28.0.1
helm install --namespace monitoring monitoring vm/victoria-metrics-k8s-stack  -f values.yaml \
    --version v0.77.0 --set grafana.adminPassword=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)

kubectl apply -f monitoring-ingress-public.yaml
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
kubectl apply -f dashboards/spegel.yaml
kubectl apply -f dashboards/node-exporter.yaml
kubectl apply -f dashboards/hpa-dashboard.yaml
kubectl apply -f dashboards/vpa-dashboard.yaml
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

TODOs:

helm install --namespace monitoring logs vm/victoria-logs-single  -f values-logs.yaml --version v0.12.4
kubectl apply -f logs-config.yaml
https://itnext.io/kubernetes-monitoring-a-complete-solution-part-9-talos-linux-system-logs-with-victorialogs-and-65c1f1e44a23

## Usefull commands:

```
kubectl -n monitoring port-forward service/vmsingle-monitoring-victoria-metrics-k8s-stack 9090:8428
kubectl -n monitoring port-forward service/vmagent-monitoring-victoria-metrics-k8s-stack 9090:8429
kubectl -n monitoring port-forward service/vmalert-monitoring-victoria-metrics-k8s-stack 9090:8080
```

TODO add as dashboard 
https://grafana.com/grafana/dashboards/15760-kubernetes-views-pods/
https://grafana.com/grafana/dashboards/21410-kubernetes-overview/


## Resources

* https://traefik.io/blog/capture-traefik-metrics-for-apps-on-kubernetes-with-prometheus/
* https://itnext.io/kubernetes-monitoring-a-complete-solution-part-3-metrics-using-the-victoria-metrics-k8s-stack-515d64b5f3ba
* https://docs.victoriametrics.com/helm/victoria-metrics-k8s-stack/
* https://itnext.io/automatically-extracting-etcd-certificates-into-a-secret-in-talos-with-prometheus-or-670047f66534
