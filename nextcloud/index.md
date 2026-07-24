# Nextcloud

To store our files, we will use nextcloud

``` bash
kubectl create namespace locker

kubectl create secret generic nextcloud-postgresql --namespace locker \
    --from-literal=username=nextcloud \
    --from-literal=password=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)

helm install nextcloud-database cnpg/cluster -f db_values.yaml --version v0.8.1 --namespace locker

# Modify to add backups. See the instructions in databases folder
kubectl apply -f db_backup.yaml

kubectl create secret generic nextcloud --namespace locker \
  --from-literal=admin-username=caretaker \
  --from-literal=admin-password=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64) \
  --from-literal=serverinfo_token=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)

kubectl apply -f storage.yaml

helm repo add nextcloud https://nextcloud.github.io/helm/
helm repo update
helm install locker nextcloud/nextcloud -f values.yaml --namespace locker --version 9.2.4

kubectl apply -f ingressroute.yaml
```

To set up the nextcloud with oidc from goauthentik follow the instructions from:

* https://integrations.goauthentik.io/chat-communication-collaboration/nextcloud/
* https://blog.cubieserver.de/2022/complete-guide-to-nextcloud-oidc-authentication-with-authentik/

It might be easier to install the apps via the terminal:

```sh
php occ app:install user_oidc
php occ app:install contacts
php occ app:install calendar
php occ app:install tasks
```

Usefull commands:

```sh
kubectl -n locker exec -it box-nextcloud-88858c579-mq7sv -- /bin/bash
su -s /bin/bash www-data
php occ app:update --all
php occ maintenance:mode --off 
php occ config:system:set overwrite.cli.url --value="https://locker.anagno.dev"
kubectl -n locker get secret nextcloud -o jsonpath="{.data.admin-password}" | base64 -d
```

https://grafana.com/grafana/dashboards/17821-nextcloud-log/
https://okxo.de/monitor-your-nextcloud-logs-for-suspicious-activities/
https://voidquark.com/blog/parsing-nextcloud-audit-logs-with-grafana-loki/

https://github.com/grafana/helm-charts/blob/main/charts/loki-stack/values.yaml