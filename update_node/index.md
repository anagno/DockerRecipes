# Mainting the nodes


NOT READY

will have to take a look at https://longhorn.io/docs/1.2.2/volumes-and-nodes/maintenance/#updating-the-node-os-or-container-runtime

maybe of interest 
https://rpi4cluster.com/scripts/undervoltage/


Then we can execute our playbook

```bash
ansible-galaxy collection install kubernetes.core
pip install kubernetes
ansible-playbook update_node/update_node.yml
```

// To check the k3s version https://update.k3s.io/v1-release/channels


## Check the use of deprecated APIs from the programs

We can use [kube-no-trouble](https://github.com/doitintl/kube-no-trouble) :

```sh
docker pull ghcr.io/doitintl/kube-no-trouble:latest
docker run -it --rm -v "${HOME}/.kube/config:/.kubeconfig" ghcr.io/doitintl/kube-no-trouble:latest -k /.kubeconfig -t v1.33.0+k3s1
```


## Usefull commands:

```
ansible all --forks 1 -b -m apt -a "autoremove=yes"
ansible aretusa -b -m apt -a "upgrade=yes update_cache=yes"
ansible all -b -m shell -a "cat /var/run/reboot-required"
ansible aretusa -m shell -a "lsblk | grep /media/storage"
```

## Check also 

https://github.com/weaveworks/kured
