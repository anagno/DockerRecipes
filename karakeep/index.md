# Self hosted bookmarks managers

```bash
kubectl create namespace bookmarks

kubectl create secret generic karakeep-oidc-secret --namespace bookmarks \
  --from-literal=client_id=ID_FROM_AUTHENTIK \
  --from-literal=client_secret=SECRET_FROM_AUTHENTIK

kubectl create secret generic karakeep --namespace bookmarks \
    --from-literal=NEXTAUTH_SECRET=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)

kubectl create secret generic karakeep-meilesearch --namespace bookmarks \
    --from-literal=MEILI_MASTER_KEY=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)

# Follow instructions in https://integrations.goauthentik.io/documentation/karakeep/
# and make certain to also use a singing key -- https://github.com/karakeep-app/karakeep/issues/516#issuecomment-2445614035

helm repo add karakeep-app https://karakeep-app.github.io/helm-charts
helm repo update
helm install --namespace bookmarks karakeep karakeep-app/karakeep -f values.yaml --version 0.33.1

# If we activate scraping the metrics
# kubectl apply -f servicemonitor.yaml

kubectl apply -f ingressroute.yaml
kubectl apply -f vpa.yaml
```

* https://integrations.goauthentik.io/documentation/karakeep/
* https://github.com/karakeep-app/karakeep/
* https://docs.karakeep.app/installation/kubernetes
* https://github.com/karakeep-app/karakeep/issues/516