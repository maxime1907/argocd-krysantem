# Requirements
## Install helm
### Linux
```bash
wget https://get.helm.sh/helm-v3.9.2-linux-amd64.tar.gz -O helm.tar.gz && tar -xvf helm.tar.gz && sudo cp linux-amd64/helm /usr/local/bin/helm && chmod +x /usr/local/bin/helm
```
### Windows
```bash
choco install kubernetes-helm
```

## Install argocd
```bash
git clone https://github.com/maxime1907/argocd-krysantem argocd
cd argocd
helm repo add argo https://argoproj.github.io/argo-helm
helm install -n argocd --create-namespace argocd argo/argo-cd --version 5.36.1 --values helm/argocd/clusters/ovh-krysantem-gra1-prod.yml --kube-context $KUBE_CONTEXT
```

## Upgrade argocd
```bash
helm upgrade -n argocd argocd argo/argo-cd --version 5.36.1 --values helm/argocd/clusters/ovh-krysantem-gra1-prod.yml --kube-context $KUBE_CONTEXT
```

# Cluster configuration
## Secrets

We use bitnami sealed secrets with our own keys that we initially have to generate

This tutorial contains parts of the blog written here:
- https://dev.to/ashokan/sealed-secrets-bring-your-own-keys-and-multi-cluster-scenario-1ee8

### Create secret keys
```bash
export PRIVATEKEY="acmetls.key"
export PUBLICKEY="acmetls.crt"
export NAMESPACE="sealed-secrets"
export CONTROLLER="sealed-secrets"
export SECRETNAME="sealed-secrets-key"

openssl req -x509 -nodes -newkey rsa:4096 -keyout "$PRIVATEKEY" -out "$PUBLICKEY" -subj "/CN=sealed-secret/O=sealed-secret"

kubectl create ns "$NAMESPACE" --context $KUBE_CONTEXT

kubectl -n "$NAMESPACE" create secret tls "$SECRETNAME" --cert="$PUBLICKEY" --key="$PRIVATEKEY" --context $KUBE_CONTEXT
```

### Encrypt a secret
```bash
export SECRET="secret.yml"
export SEALED_SECRET="sealed-secret.yml"
export SCOPE="cluster-wide"

kubeseal \
    --controller-name $CONTROLLER \
    --controller-namespace $NAMESPACE \
    --secret-file $SECRET \
    --sealed-secret-file $SEALED_SECRET \
    --scope cluster-wide \
    --context $KUBE_CONTEXT
```
