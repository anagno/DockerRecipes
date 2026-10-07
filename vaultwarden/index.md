# Self hosted password manager

```bash
kubectl create namespace vault

kubectl create secret generic vaultwarden-postgresql --namespace vault \
    --from-literal=username=vaultwarden \
    --from-literal=password=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)

helm install vaultwarden-database cnpg/cluster -f db_values.yaml --version v0.8.1 --namespace vault

# Modify to add backups. See the instructions in databases folder
kubectl apply -f db_backup.yaml

kubectl apply -f pvc.yaml

kubectl create secret generic vaultwarden-admin --namespace vault \
  --from-literal=admin-password=$(head -c 512 /dev/urandom | LC_CTYPE=C tr -cd 'a-zA-Z0-9' | head -c 64)

# Read instructions from here: https://integrations.goauthentik.io/security/vaultwarden/

kubectl create secret generic vaultwarden-oidc-secret --namespace vault \
  --from-literal=client_id=ID_FROM_AUTHENTIK \
  --from-literal=client_secret=SECRET_FROM_AUTHENTIK

helm repo add vaultwarden https://guerzon.github.io/vaultwarden
helm repo update
helm install --namespace vault vaultwarden vaultwarden/vaultwarden -f values.yaml --version v0.46.3

kubectl apply -f ingressroute.yaml

```
