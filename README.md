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
