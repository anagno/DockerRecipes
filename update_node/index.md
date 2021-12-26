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


# Updating k3s
https://rancher.com/docs/k3s/latest/en/upgrades/