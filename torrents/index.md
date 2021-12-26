Just a small torrent server with the necessary samba server to access the data

```
kubectl create namespace torrents
kubectl apply -f pvc.yaml
kubectl apply -f torrents.yaml
kubectl apply -f ingressroute.yaml
```

After the deployment I should remember to add the black list 
https://github.com/sayomelu/transmission-blocklist/raw/release/blocklist.gz
of ips in the torrents


## Resources:

* https://github.com/sayo-melu/transmission-blocklist
* https://gist.github.com/shmup/29566c5268569069c256