A small rss server for keeping up with the rss feeds

```bash
kubectl create namespace news

kubectl create secret generic freshrss-postgresql --namespace news \
    --from-literal=username=freshrss \
    --from-literal=password=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)

helm install freshrss-database cnpg/cluster -f db_values.yaml --version v0.6.0 --namespace news

kubectl create secret generic freshrss-oidc-secret --namespace news \
  --from-literal=client_id=ID_FROM_AUTHENTIK \
  --from-literal=client_secret=SECRET_FROM_AUTHENTIK \
  --from-literal=crypto_key=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)

# Follow the instructions to setup the authentik: https://freshrss.github.io/FreshRSS/en/admins/16_OpenID-Connect-Authentik.html

kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingressroute.yaml
```

# Usefull commands
```bash
kubectl -n news exec -it freshrss-88b8985bb-5lrgt -- bash
echo "<?php phpinfo();" >> p/i/phpinfo.php


# To move the subscriptions 
scp ubuntu@old_freshrss:/home/ubuntu/freshrss/www/freshrss/data/users/anagno/db.sqlite db.sqlite
kubectl cp db.sqlite news/freshrss-56d767678c-md6h9:/var/www/FreshRSS/data/users/anagno/db.sqlite


./cli/do-install.php --default_user caretaker --auth_type http_auth --environment production --base_url https://news.anagno.dev --language en --title FreshRSS --api_enabled --db-type sqlite

# To copy a local config.php to the container
kubectl cp config.php news/freshrss-795c9d8b6c-n89rr:/data/config.php
```

!!!Note
    The first time configuration must be done automatically. I could not locate
    a way to automate it. Remember also to activate the api and remember to change
    the config to use the getenv('FRESHRSS_POSTGRESQL__USER')


## Resources:

* https://github.com/t13a/helm-chart-freshrss/blob/master/values.yaml
* https://docs.syseleven.de/metakube/de/tutorials/sharing-volumes-between-pods
* https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/

## TODOs:

* https://github.com/FreshRSS/FreshRSS/pull/5748
* https://github.com/FreshRSS/FreshRSS/issues/6551