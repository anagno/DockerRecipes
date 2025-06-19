# Dynamic IP

In my home network I do not have a static IP. So I have to be able to update my domain names to point
to my dynamic IP. This is necessary to be able to access the services of my cluster from outside my network.

!!! note
    It is also necessary to set up forwarding rules to your router, to forward the requests to the cluster.


For the purpose I have created a [docker image](https://github.com/anagno/gandi-ddns) that is using my dns provider
api to update my IP when it changes.

Furthermore we are going to use [external-dns](https://github.com/kubernetes-sigs/external-dns) to automatically update our dns.


``` bash
kubectl create namespace general
kubectl -n general create secret generic gandi --from-literal=API_KEY=GANDITOKEN
kubectl apply -f ddns-anagno-me.yaml
kubectl apply -f ddns-anagno-dev.yaml

helm repo add external-dns https://kubernetes-sigs.github.io/external-dns/ 
helm repo update
helm install external-dns external-dns/external-dns -f values.yaml --namespace general --version 1.17.0

kubect apply -f vpa.yml

kubectl apply -f test.yaml
# test that everything works
kubectl delete -f test.yaml
```

!!!Note
    Initially the prometheus monitoring should not be activate until we deploy the 
    monitoring stack. Afterwards we can activate it.

## Usefull commands
```
kubectl --namespace general logs -f -l "app=ddns"
```
!!! note
    Take a look at: https://docs.k8s-at-home.com/guides/dyndns/#creating-a-secret


## Resources

* https://github.com/kubernetes-sigs/external-dns
* https://github.com/kubernetes-sigs/external-dns/issues/1394#issuecomment-585228684
* https://github.com/kubernetes-sigs/external-dns/tree/438d06f3c45cf66d08945ae18d17e29c540d5c96/charts/external-dns
* https://grafana.com/grafana/dashboards/15038-external-dns/