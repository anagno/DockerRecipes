# Managing databases in Kubernetes

More or less after the [deprecation](https://github.com/bitnami/containers/issues/83267)
of the bitnami charts, I was faced with a lot of problems with my databases. To avoid a
similar incident in the future, I prefer to start deploying my own databases in the cluster
and for achieving it that I prefer to have an operator that will make things easier.

## cloudnative-pg

Since most of my deployments were postgresl DBs I will go with a operator for the 
[PostgreSQL](https://cloudnative-pg.io/documentation/current/). This operator seems to be 
the most mature and has a lot of contributors so it shoud be future proof.



``` bash
helm repo add cnpg https://cloudnative-pg.github.io/charts
helm repo update
kubectl create namespace databases
helm install cnpg cnpg/cloudnative-pg -f operator_values.yml --version v0.29.0 --namespace databases

kubectl apply -f snapshots.yml
```

We will be also using a special db storage class for the databases based 
on the [suggestions](https://cloudnative-pg.io/documentation/current/storage/#block-storage-considerations-cephlonghorn)
of the operator:

```bash
kubectl apply -f db_storage.yml
```

After the operator is deployed we can create databses. We provide 2 examples.
One for highly available cluster:

``` bash
kubectl create namespace test-database

kubectl create secret generic db-user-pass --namespace test-database \
    --from-literal=username=app \
    --from-literal=password=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)

helm install database cnpg/cluster -f ha_db_values.yml --version v0.8.1 --namespace test-database
```
The high availability one, does not replicate the data on the level of longhorn, but each database
has their own non duplicate volume and they sync the data between the instances of the databases

The "normal" not highly available databases:


``` bash
kubectl create namespace test-database

kubectl create secret basic-auth db-user-pass --namespace test-database \
    --from-literal=username=app \
    --from-literal=password=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)

helm install database cnpg/cluster -f non_ha_db_values.yml --version v0.8.1 --namespace test-database
```

The non high availability example just has one instance of the database and uses longhorn
for replication of the data.

## Backup for databases

Currently it is not possible to add automated backups via the [helm charts](https://github.com/cloudnative-pg/charts/pull/359).
But the cloudnative-pg supports backups via snapshots. So to work around it we will do it manually.

We will have to change the definition of the Cluster manually. For the examples above we have to execute:

```bash
KUBE_EDITOR="nano" kubectl -n test-database edit cluster database-cluster
```

and add in the definition of the cluster: 

```bash
  backup:
    volumeSnapshot:
       className: default-snapshot-class
```

To trigger immediately a backup we can use the `backup.yaml`. In other database we have to change `    name: database-cluster`
to the right name.


## kubectl plugin

Instructions from the [official site](https://cloudnative-pg.io/documentation/current/kubectl-plugin/): 

```bash
curl -sSfL \
  https://github.com/cloudnative-pg/cloudnative-pg/raw/main/hack/install-cnpg-plugin.sh | \
  sudo sh -s -- -b /usr/local/bin
```

## Resources
* https://medium.com/@camphul/cloudnative-pg-in-the-homelab-with-longhorn-b08c40b85384
* https://cloudnative-pg.io/documentation/current/kubectl-plugin/