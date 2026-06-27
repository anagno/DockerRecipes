# Reverse Proxy

Since we only have a single public IPv4, if we want to set up multiple public services we 
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
helm install -n proxy traefik traefik/traefik -f traefik-values.yaml --version 41.0.0
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
kubectl apply -f security_headers.yaml
```

Traefik will be my main proxy that will forward requests to other services. So 
we also deploy these requirements:

```bash
kubectl apply -f grigoris_proxy.yaml
```

## Cert-manager

Since we want high availability, we need also to handle our certificates in a way that can be 
hanlded from traefik. Traefik offers this support in the enteprise version, but since I am 
cheap, I will use an another manager (i.e. cert-manager)


```bash
kubectl create namespace cert-manager
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager --namespace cert-manager -f cert-values.yaml --version v1.20.3
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

kubectl apply -f traefik-dashboard.yaml
```

## Testing that everything works

To test that everything works, we can deploy a testing whoami service:

```bash
kubectl apply -f whoami.yaml
# Check that everything works and the delete the service
kubectl delete -f whoami.yaml
# Forcing renew. Good practice when updating the cert-manager to make sure that
# everything still works
kubectl cert-manager status certificate test.anagno.dev -n general
kubectl cert-manager renew test.anagno.dev -n authentication
```

## Gateway API vs Ingress

The Gateway API is the next generation of Kubernetes Ingress and it was investigate to see if it should
be used in the deployment of the services. From the experimentations I did unfortunatelly, it did not fit
yet my uses cases (see gateway_api folder). 

The main problem I faced was that for each Gateway I could only have a single certificate
to decrypt the HTTPS traffic. That means that I would need to either setup
* [DNS01](https://cert-manager.io/docs/configuration/acme/dns01/acme-dns/) resolver for cert-manager 
* or use supbaths.

The DNS resolver would introduce that I would have an extra service like cloudflare or depend on obsolete 
plugins of cert-manager. Both not ideal for the moment. 

The subpaths could lead to complexities with the deployed services. Although most services, have the 
possibility to adapt the subpath, some might expect to be on the root of the domain name.

Alternative for each subdomain I could create a new Gateway Kubernetes Object, but that result 
in more complicated setups.

In the future there might be better support of my uses and it might be worth it switching to it. 

The problems I faced:

* Each Gateway can have only one certificate to decrypt the HTTPS traffik. Pontial solutions for this are:

    * https://github.com/kubernetes-sigs/gateway-api/issues/3249
    * https://github.com/kubernetes-sigs/gateway-api/issues/2111
    * https://github.com/kubernetes-sigs/gateway-api/discussions/3418
    * https://gateway-api.sigs.k8s.io/geps/gep-1713/
    * https://github.com/kubernetes-sigs/gateway-api/issues/1713

  The most promising solution probably is the `ListenerSet`s, but that is not still
  supported in Traefik.
* The TLSRoute does not support externalName services. The TLSRoute will be used in `grigoris_proxy`

* The Middlewares of traefik can not still be defined in the Gateway API. So we still need the 
specific CDRs from Traefik. Pontential solution for this one is:
    * https://gateway-api.sigs.k8s.io/geps/gep-1767/

* The authentik has only experimental support for the Gateway API


Since there no plan yet to deprecate the [Ingress API](https://gateway-api.sigs.k8s.io/faq/#will-gateway-api-replace-the-ingress-api),
I will remain for the moment with the Ingress API logic


Resources:

* https://medium.com/dev-genius/setup-traefik-v2-for-ha-on-kubernetes-20311204fa6f
* https://traefik.io/blog/install-and-configure-traefik-with-helm/
* https://kubernetes.github.io/ingress-nginx/deploy/baremetal/
* https://www.thebookofjoel.com/k3s-cert-manager-letsencrypt
* https://cert-manager.io/docs/usage/kubectl-plugin/