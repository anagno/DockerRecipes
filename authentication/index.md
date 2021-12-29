We will use OpenLDAP to define our users only once, and be able to share the authoriyation
of the users across the the services. Furthermore we will use authelia to obtain a second factor
for the user and to protect better our services.

```
kubectl create namespace authentication

kubectl create secret generic openldap-admin --namespace authentication \
  --from-literal=admin-password=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64) \
  --from-literal=config-password=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)

kubectl create secret generic openldap-readonly --namespace authentication \
  --from-literal=readonly-password=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)

# We want this secret to be available everywhere
kubectl annotate secret openldap-readonly kubed.appscode.com/sync="" -n authentication

kubectl apply -f ldap-storage.yaml
kubectl apply -f ldap.yaml
```

To activate hashing of our passwords we have to enable it in the openldap server
```
#kubectl -n authentication logs -f -l "app=openldap"
# To retrieve the name of the pod
kubectl -n authentication get pods -l "app=openldap"
# To enter the pod
kubectl -n authentication exec -it openldap-7b8556bcb5-pwg2n -- bash

cd
cat <<EOF | tee load-ppolicy.ldif
dn: cn=module{0},cn=config
changetype: modify
add: olcModuleLoad
olcModuleLoad: ppolicy
EOF

cat <<EOF | tee overlady-ppolicy.ldif
dn: olcOverlay={2}ppolicy,olcDatabase={1}mdb,cn=config
objectClass: olcOverlayConfig
objectClass: olcPPolicyConfig
olcOverlay: {2}ppolicy
olcPPolicyHashCleartext: TRUE
EOF

ldapadd -Q -Y EXTERNAL -H ldapi:/// -f load-ppolicy.ldif 
ldapadd -Q -Y EXTERNAL -H ldapi:/// -f overlady-ppolicy.ldif 

```

For managing our ldap server we are going to use the phpLDAPadmin docker image.
We can now create all the necessary data, that will also be used from authelia

```
kubectl apply -f users.yaml
```

Now we can deploy authelia

```
kubectl create secret generic authelia --namespace authentication \
  --from-literal=session=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64) \
  --from-literal=jwt-secret=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64) \
  --from-literal=encryption-key=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)

kubectl create secret generic authelia-mariadb --namespace authentication \
  --from-literal=root-password=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64) \
  --from-literal=authelia-password=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)

kubectl apply -f authelia-storage.yaml
kubectl apply -f authelia-redis.yaml
kubectl apply -f authelia-mariadb.yaml
kubectl apply -f authelia.yaml
kubectl apply -f authelia-ingressroute.yaml
kubectl apply -f authorization-middleware.yaml
kubectl apply -f authorization-basic-middleware.yaml
```

And now we can also make publicly from the interenet some dashboards,
now that we can limit their access

```
kubectl apply -f users-public.yaml
kubectl apply -f proxy-public.yaml
kubectl apply -f storage-public.yaml
```


Resources:

* https://github.com/osixia/docker-openldap/issues/347
* https://github.com/osixia/docker-openldap/issues/303
* https://tobru.ch/openldap-password-policy-overlay/
* https://github.com/osixia/docker-openldap/pull/333
* How to add organisazations units, etc: https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-a-basic-ldap-server-on-an-ubuntu-12-04-vps#AddOrganizationalUnits,Groups,andUsers 
* http://www.zytrax.com/books/ldap/
* http://www.openldap.org/doc/admin24/
* https://github.com/osixia/docker-openldap
* https://geek-cookbook.funkypenguin.co.nz/recipes/openldap/