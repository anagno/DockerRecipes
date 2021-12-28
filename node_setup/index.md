# Node preparation

Before even starting with setting up the cluster, we should prepare the 
nodes. These steps should be repeated for all the nodes of the cluster.

## Basic settings on each node

I am going to use Ansible for most tasks, but before we are able to do that we 
will have to a couple of manual tasks.

### Preparing the hosts file of Ansible

In my internal logic of the ansible scripts, I have 3 groups:

* masters -- which eventually will be the master nodes of the k3s
* workers -- which eventually will be the "worker" nodes of the k3s
* disks -- special nodes that also have an hhd/ssd disk for writing bigger data

!!!Note
      In the disks group we are also defining the path of the mounted disks.
      It is usefull when we will be setting up our storage

So your hosts file will look like something:

```yaml
all:
  hosts:
    node_1:
      ansible_host: whatever
      ansible_user: whatever
      ansible_ssh_pass: whatever
    node_2:
      ansible_host: whatever
      ansible_user: whatever
      ansible_ssh_pass: whatever
    node_3:
      ansible_host: whatever
      ansible_user: whatever
      ansible_ssh_pass: whatever
  children:
    masters:
      hosts:
        node_1:
    workers:
      hosts:
        node_2:
        node_3:
    disks:
      hosts:
        node_3:
          disk_path: /dev/sda
          UUID: whatever
        node_4:
          disk_path: /dev/sdb
          UUID: whatever
```

Using the ```ansible_ssh_pass``` variable is not the best idea, but if you insist
it is better to encrypt the data by using the ansible-vault. This can be achieved by

```bash
ansible-vault encrypt_string 'my_strong_password' --name 'ansible_ssh_pass' 
```

!!! note
    The above command does not pass the vault_password_file because it is 
    defined in the ansible.cfg

### Operating system 

The most simple OS to use is [Ubuntu](https://ubuntu.com/download/raspberry-pi). 
You should use a 64bit image, because generally many of the images we are using are 
only available for aarch64 not the arm4vl (which is the 32 bit version).

Ubuntu is bootable from a usb [from Ubuntu 20.10](https://forums.raspberrypi.com/viewtopic.php?t=295609)
So we will have to make it [bootable](https://forums.raspberrypi.com/viewtopic.php?f=131&t=278791):


!!!Note
    Using the latest version (as of the writting 21.10) is not possible because the `open-iscsi` was broken.
    The kernel did not have the `iscsi_tcp` module. This could be related to the fact that Ubuntu
    moved some upstream packages to a [seperate package](https://github.com/k3s-io/k3s/issues/4188#issuecomment-982503626)

### Upgrade the firmware (if necessary)

We are going to use USB disk, instead of SD cards. But the firmware of Raspberry Pi
support the boot from the USB after 2021. So make certain the firmware is updated.
Check the appendix for details on how to update the firmware

### Set up the password and hostname

We have to SSH log in manually to every node, and change the password when asked (use 
strong passwords). So for each node we will have the username ubuntu and the password
you used.

After that step we can restart the node and everything else will be done via ansible

## First setup of via ansible 

The scripts we have in ansible makes it easy. Just execute:

```bash
ansible-playbook node_setup/setup_node.yml
```

To set-up a signle node we can execute:

```bash
ansible-playbook node_setup/setup_node.yml --limit "node"
```

## Resources

* https://opensource.com/article/20/6/kubernetes-raspberry-pi
* https://askubuntu.com/questions/1189480/raspberry-pi-4-ubuntu-19-10-cannot-enable-cgroup-memory-at-boostrap
* https://github.com/kubernetes/kubeadm/blob/master/docs/ha-considerations.md#kube-vip
* https://thebsdbox.co.uk/2020/01/02/Designing-Building-HA-bare-metal-Kubernetes-cluster/#Networking-load-balancing
* https://tanzu.vmware.com/content/blog/exploring-kube-apiserver-load-balancers-for-on-premises-kubernetes-clusters
* https://icicimov.github.io/blog/kubernetes/Kubernetes-cluster-step-by-step-Part5/
* https://stackoverflow.com/questions/56634139/how-to-use-the-master-node-as-worker-node-in-kubernetes-cluster
* https://www.shogan.co.uk/kubernetes/building-a-pi-kubernetes-cluster-part-3-worker-nodes-and-metallb/
* https://opensource.com/article/20/7/homelab-metallb
* https://blog.lzoledziewski.pl/2020/06/01/noctua-5v-pwm-fan-rpi-4/
* https://blog.driftking.tw/en/2019/11/Using-Raspberry-Pi-to-Control-a-PWM-Fan-and-Monitor-its-Speed/
* https://www.raspberrypi.org/forums/viewtopic.php?f=63&t=244194
* https://gist.github.com/alwynallan/1c13096c4cd675f38405702e89e0c536
* https://github.com/geerlingguy/raspberry-pi-dramble
* https://github.com/k3s-io/k3s



