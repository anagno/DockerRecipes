A small calibre server for accessing my books

```
kubectl create namespace library
kubectl apply -f pvc.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingressroute.yaml
kubectl apply -f smb.yaml
```

!!!Note
    The first time configuration must be done automatically. I could not locate
    a way to automate it. Furthermore the proxy authentication is not perfect.
    We have to first create manually the users in the database of 
    [calibre-web](https://github.com/janeczku/calibre-web/wiki/Setup-Reverse-Proxy/#traefik--241-with-authelia-forward-auth)
    Furthermore, for the opds catalogue to work, we have to activate the anonymous
    browsing from the menu. Probably a bug in the calibre-web.
    Theoretically it should not be a problem, since we are behind our proxy and authentication


## Usefull commands 

```bash
nohup rsync -avh --progress --bwlimit=10000 /home/anagno/Books/ . > nohup2.out 2>nohup2.err < /dev/null &
mount -t cifs -o username=anagno,password=anagno //192.168.179.236/books/ smb_books/
sudo mount -t cifs -o username=anagno,password=anagno //192.168.179.236/books/ smb_books/
cd smb_books/
sudo rsync -avh --progress --bwlimit=10000 /home/anagno/Books/ .
sudo umount smb_books 
rm -R smb_books/
```


## Resources:

* https://github.com/janeczku/calibre-web/wiki/Setup-Reverse-Proxy/#reverse-proxy
* https://superuser.com/questions/1229146/rsync-for-syncing-4tb-from-smb-to-usb-volume-hostnameproblem