# Dynamic IP

In my home network I do not have a static IP. So I have to be able to update my domain names to point
to my dynamic IP. This is necessary to be able to access the services of my cluster from outside my network.

!!! note
    It is also necessary to set up forwarding rules to your router, to forward the requests to the cluster.


For the purpose I have created a [docker image](https://github.com/anagno/gandi-ddns) that is using my dns provider
api to update my IP when it changes.

``` bash
kubectl create namespace general
kubectl -n general create secret generic gandi --from-literal=API_KEY=GANDITOKEN
kubectl apply -f ddns-anagno-me.yaml
kubectl apply -f ddns-anagno-dev.yaml
```

and the ddns.yaml is:

``` yaml
--8<-- "ddns/ddns-anagno-me.yaml"
```

# Usefull commands
```
kubectl --namespace general logs -f -l "app=ddns"
```
!!! note
    Take a look at: https://docs.k8s-at-home.com/guides/dyndns/#creating-a-secret