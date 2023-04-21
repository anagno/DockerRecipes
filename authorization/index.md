

We will use [Authentik](https://goauthentik.io/). Authentik
allows us to define all the necessary permissions and have a second factor for the 
user and to protect better our services.

```
kubectl create namespace authorization

kubectl create secret generic authentik --namespace authorization \
  --from-literal=secret-key=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64) \
  --from-literal=ak-admin-pass=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)

kubectl create secret generic authentik-database --namespace authorization \
  --from-literal=postgresql-postgres-password=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64) \
  --from-literal=postgresql-password=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)

kubectl create secret generic authentik-redis --namespace authorization \
  --from-literal=redis-password=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64) 

kubectl apply -f authentik-storage.yaml

We have to create first the certificates, so we can also use them inside 
authentik. 

kubectl apply -f authentik-ingressroute.yaml

helm repo add authentik https://charts.goauthentik.io
helm repo update
helm install --namespace authorization authentik authentik/authentik -f values.yaml --version 2023.4.1
```

After the initialization we should also deploy the `recovery-email-verification.yaml` and the  
the [two-factor login](https://goauthentik.io/docs/flow/examples/flows#two-factor-login). Pay attention to force it.


And now we can also make publicly from the interenet some dashboards,
now that we can limit their access. To do that we will have first to define them inside the authentik app.

We will have to create a new Provider (usually the [proxy provider](https://goauthentik.io/docs/providers/proxy/)) 
and a new [Application](https://goauthentik.io/docs/applications) for the proxy and the storage and then we 
can deploy the ingress routes.


```
kubectl apply -f vpa.yaml
kubectl apply -f sites/proxy-public.yaml
kubectl apply -f sites/storage-public.yaml
```

For some of the proxy prover we need the extra middleware, because the we need to forward extra headers

```
kubectl apply -f authentik-middleware-remote-user-header.yaml
```

For the moment there is no good way of doing this via command line, but there might be progress in the future:

* https://github.com/goauthentik/helm/issues/127

## Usefull commands 

```
kubectl -n authorization get secret authentik -o jsonpath="{.data.ak-admin-pass}" | base64 -d

# https://goauthentik.io/developer-docs/blueprints/export#global-export
kubectl -n authorization exec -it authentik-worker-65c8449ccc-9nkdb -- bash
ak export_blueprint


```


## Resources:

* https://goauthentik.io/docs/installation/kubernetes
* https://goauthentik.io/docs/providers/proxy/
* https://goauthentik.io/docs/policies/expression?utm_source=authentik
* https://goauthentik.io/docs/flow/
