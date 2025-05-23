# Reverse Proxy

Since we only have a single public IP, if we want to set up multiple public services we 
will need a reverse proxy to redirect the requests to the correct services. Our proxy
will also handle the ssl termination, to simplify the set up of the services. 

For managing our certificates we will be using `cert-manager` and as a proxy we 
will be using `Traefik`.

!!! note
    Traefik is the default proxy from k3s, but to have more granular control
    we are disabling it and we are going to deploy it as Helm chart

## Traefik

Similarly, to deploy Traefik we have to execute:

```bash
kubectl create namespace proxy
helm repo add traefik https://helm.traefik.io/traefik
helm repo update
helm install -n proxy traefik traefik/traefik -f traefik-values.yaml --version 35.2.0
```

!!! note
    In the logs we might have a warning and we might have to increase the rmeem_max
    `ansible all -b -m shell -a "sysctl -w net.core.rmem_max=2500000"`. To retrieve
    the logs do `kubectl -n proxy logs -f -l "app.kubernetes.io/name=traefik"`

``` yaml
--8<-- "proxy/traefik-values.yaml"
```

Now that our proxy is running we can define some Middlewares to simplify the 
deployment of services:

```bash
kubectl apply -f https_redirect.yaml
kubectl apply -f security_headers.yaml
```

Traefik will be my main proxy that will forward requests to other services. So 
we also deploy these requirements:

```bash
kubectl apply -f grigoris_proxy.yaml
kubectl apply -f box_proxy.yaml
kubectl apply -f traefik-dashboard.yaml
```

## Cert-manager

Since we want high availability, we need also to handle our certificates in a way that can be 
hanlded from traefik. Traefik offers this support in the enteprise version, but since I am 
cheap, I will use an another manager (i.e. cert-manager)


```bash
kubectl create namespace cert-manager
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager --namespace cert-manager -f cert-values.yaml --version v1.17.2
kubectl apply -f cert-vpa.yaml
```


With `cert-values.yaml`:

``` yaml
--8<-- "proxy/cert-values.yaml"
```

!!!Note
    Initially the prometheus monitoring should not be activate until we deploy the 
    monitoring stack. Afterwards we can activate it.

Now that the `cert-manager` is running we can create our certificate issuers:

```bash
kubectl apply -f self_signed.yaml
kubectl apply -f letenrcypt_stagging.yaml
kubectl apply -f letenrcypt.yaml
```

!!!Note
    We are using `traefik-cert-manager` as name for the traefik ingress.
    That we will have to specify it when we deploy the traefik

## Testing that everything works

To test that everything works, we can deploy a testing whoami service:

```bash
kubectl apply -f whoami.yaml
# Check that everything works and the delete the service
kubectl delete -f whoami.yaml
# Forcing renew. Good practice when updating the cert-manager to make sure that
# everything still works
kubectl cert-manager status certificate login.anagno.me -n authentication
kubectl cert-manager renew login.anagno.me -n authentication
```

Resources:

* https://medium.com/dev-genius/setup-traefik-v2-for-ha-on-kubernetes-20311204fa6f
* https://traefik.io/blog/install-and-configure-traefik-with-helm/
* https://kubernetes.github.io/ingress-nginx/deploy/baremetal/
* https://www.thebookofjoel.com/k3s-cert-manager-letsencrypt
* https://cert-manager.io/docs/usage/kubectl-plugin/