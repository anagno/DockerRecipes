# Nextcloud

To store our files, we will use nextcloud

``` bash
kubectl create namespace cyberlocker

kubectl create secret generic nextcloud --namespace cyberlocker \
  --from-literal=admin-username=caretaker \
  --from-literal=admin-password=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64) \
  --from-literal=serverinfo_token=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64) \
  --from-literal=smtp_username= \
  --from-literal=smtp_password= \
  --from-literal=smtp_host=

kubectl create secret generic nextcloud-postgresql-database --namespace cyberlocker \
  --from-literal=postgres-password=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64) \
  --from-literal=password=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)

kubectl create secret generic nextcloud-redis --namespace cyberlocker \
  --from-literal=redis-password=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64) 

kubectl apply -f storage.yaml


helm repo add nextcloud https://nextcloud.github.io/helm/
helm repo update
helm install cyberlocker nextcloud/nextcloud -f values.yaml --namespace cyberlocker --version 4.6.6

kubectl apply -f ingressroute.yaml
kubectl apply -f vpa.yaml
```

To set up the nextcloud with oidc from goauthentik follow the instructions from:

* https://blog.cubieserver.de/2022/complete-guide-to-nextcloud-oidc-authentication-with-authentik/


Usefull commands:

```sh
kubectl -n cyberlocker exec -it box-nextcloud-88858c579-mq7sv -- /bin/bash
su -s /bin/bash www-data
php occ app:update --all
php occ maintenance:mode --off 
php occ config:system:set overwrite.cli.url --value="https://cyberlocker.anagno.dev"

kubectl -n cyberlocker get secret nextcloud -o jsonpath="{.data.admin-password}" | base64 -d
kubectl -n cyberlocker exec -it cyberlocker-postgresql-0 -- psql -d nextcloud -U nextcloud
```


https://grafana.com/grafana/dashboards/17821-nextcloud-log/
https://okxo.de/monitor-your-nextcloud-logs-for-suspicious-activities/
https://voidquark.com/blog/parsing-nextcloud-audit-logs-with-grafana-loki/

https://github.com/grafana/helm-charts/blob/main/charts/loki-stack/values.yaml