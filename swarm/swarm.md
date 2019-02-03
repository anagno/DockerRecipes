## Setting up the Raspberry Pi as a Ceph client.

?> **Tip** Read first the [Persistent storage]() section

Log in the Raspberry Pi by using the 

```bash
sudo apt-get install ceph-common
sudo mkdir /mnt/storage
sudo nano /etc/fstab
```

aphrodite.lan:6789,artemis.lan:6789,hephaestus.lan:6789:/ /mnt/storage ceph name=swarm,secret=AQBUudVbKCn4MRAA+gHPMqU5AdeE/mipjch/iA==,noatime,_netdev 0 0

In any docker manager execute:

```
docker node update --label-add persistence=ceph NAME_OF_NEW_NODE

docker node ls -q | xargs docker node inspect \
  -f '{{ .ID }} [{{ .Description.Hostname }}]: {{ .Spec.Labels }}'

docker node ls -q | xargs docker node inspect \
  -f '{{ .ID }} [{{ .Description.Hostname }}]: {{ .Description.Platform }}'
```


