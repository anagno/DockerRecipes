A small rss server for keeping up with the rss feeds

```bash
kubectl create namespace news
kubectl apply -f pvc.yaml
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
kubectl cp db.sqlite news/freshrss-6fb94cb5d-gzsnr:/var/www/FreshRSS/data/users/anagno/db.sqlite

# To copy a local config.php to the container
kubectl cp config.php news/freshrss-7495bcb86b-5wtd7:/var/www/FreshRSS/data/config.php
```

!!!Note
    The first time configuration must be done automatically. I could not locate
    a way to automate it. Remember also to activate the api


## Resources:

* https://github.com/t13a/helm-chart-freshrss/blob/master/values.yaml
* https://docs.syseleven.de/metakube/de/tutorials/sharing-volumes-between-pods
* https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/