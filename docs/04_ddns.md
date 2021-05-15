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
kubectl apply -f ddns.yaml
```

and the ddns.yaml is:

``` yaml
apiVersion: apps/v1 
kind: Deployment
metadata:
  namespace: general
  name: ddns-updater
spec:
  selector:
    matchLabels:
      app: ddns
  replicas: 1
  template:
    metadata:
      labels:
        app: ddns
    spec:
      containers:
      - name: ddns
        image: anagno/gandi-ddns:0.3
        resources:
          limits:
            #cpu: 100m
            memory: 10Mi
        env:
        - name: DOMAIN
          value: "example.com"
        - name: SUBDOMAIN
          value: "subdomain"
        - name: APIKEY
          valueFrom:
            secretKeyRef:
              name: gandi
              key: API_KEY
      dnsPolicy: Default
```