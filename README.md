# PREREQUISITO - INSTALAR CERT MANAGER

kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.crds.yaml
helm repo add jetstack https://charts.jetstack.io
helm repo update

helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace

  

# Generar password aleatorio
pass=$(openssl rand -base64 48 | tr -d "=+/" | head -c 32)
echo "Password: $pass"

# Generar hash bcrypt (requiere htpasswd)
hashed_pass=$(htpasswd -bnBC 10 "" "$pass" | tr -d ':\n')
echo "Password Hash: $hashed_pass"

# Generar signing key
signing_key=$(openssl rand -base64 48 | tr -d "=+/" | head -c 32)
echo "Signing Key: $signing_key"

helm install kargo \
  oci://ghcr.io/akuity/kargo-charts/kargo \
  --namespace kargo \
  --create-namespace \
  --set api.adminAccount.passwordHash="$hashed_pass" \
  --set api.adminAccount.tokenSigningKey="$signing_key" \
  --wait

## al servicio me ha dado error
kubectl port-forward -n kargo pod/kargo-api-5b78467dcb-klqth 3000:8080



# eliminar
helm uninstall kargo -n kargo

