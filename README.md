# Gateway API Presentation - DevoXX France 2025

## Set up the demos

```bash
# Set up K3S  cluster
# Disable the by-default Traefik installation
k3d cluster create devoxx --port 80:80@loadbalancer --port 443:443@loadbalancer --port 8080:8080@loadbalancer --port 9090:9090@loadbalancer --k3s-arg "--disable=traefik@server:0"

# Install the experimental Gateway API CRDs (not installed with traefik)
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.2.1/experimental-install.yaml

# Install Traefik with Gateway API enabled
# With no Gateway by default
# With the experimental channel enabled
helm repo add --force-update traefik https://traefik.github.io/charts
helm upgrade --install --create-namespace --namespace traefik traefik traefik/traefik -f ./traefik_values.yaml
```

# ###############
# First backend
# ###############

```bash
# Deploy the manifests
kubectl apply -f manifests/01-first-route

# Reach the backend
# The header x-devoss has been added

curl https://whoami.docker.localhost/
```
# ###################
# Second backend
# Cross-Namespace
# BackendTLSPolicy
# ###################

```bash
# Create namespaces
kubectl create namespace routes
kubectl label namespace routes type=tls-routes --o
verwrite

kubectl create namespace whoami

# Create application namespace
# Store certificates
kubectl create secret tls external-certs --namespace routes --cert=./certs/external-crt.pem --key=./certs/external-key.pem
kubectl create secret tls internal-certs --namespace whoami --cert=./certs/internal-crt.pem --key=./certs/internal-key.pem

# Store the CA Root
kubectl create configmap internal-ca --namespace whoami --from-file ca.crt=./certs/internal-rootCA.pem

# Deploy the manifests
kubectl apply -f manifests/02-refgrant-backentlspolicy

# Reach the TLS backend
curl https://whoami-tls.docker.localhost/whoami
```