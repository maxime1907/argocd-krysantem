# argocd-krysantem

ArgoCD configuration files for krysantem

## Services configuration

```bash
kubectl apply -f cert-manager/applicationset.yml -n argocd
```

## Cluster installation single node with k0s calico containerd coreos openebs csi

### Install
```bash
curl -sSLf https://get.k0s.sh | sudo sh
```

### Generate k0s config file with default storage provider
- https://docs.k0sproject.io/v1.23.8+k0s.0/storage/
- https://docs.k0sproject.io/v1.23.8+k0s.0/configuration/

### Install k0s cluster
```bash
sudo k0s install controller --single
sudo k0s start
```

### Wait for node to come up online
```bash
watch -n 5 sudo k0s kubectl get nodes
```

### Get cluster admin kubeconfig
```bash
sudo k0s kubeconfig admin
```

### Configure remote access on your desktop
```bash
mkdir -p ~/.kube
```
### Paste the output of cluster admin kubeconfig inside
```bash
nano ~/.kube/config
```

## Cluster initialization
###  Install helm
#### Linux
```bash
wget https://get.helm.sh/helm-v3.9.2-linux-amd64.tar.gz -O helm.tar.gz && tar -xvf helm.tar.gz && sudo cp linux-amd64/helm /usr/local/bin/helm && chmod +x /usr/local/bin/helm
```
##### Windows
```bash
choco install kubernetes-helm
```

### Install argocd
```bash
git clone https://github.com/maxime1907/argocd-krysantem argocd
cd argocd
helm repo add argo https://argoproj.github.io/argo-helm
helm install -n argocd --create-namespace argocd argo/argo-cd --version 5.27.3 --values argocd/clusters/k0s-krysantem.yml
```

### Install openebs
```bash
kubectl apply -f openebs/applicationset.yml -n argocd
```

## Cluster configuration
### Secrets
We use bitnami sealed secrets with our own keys that we initially have to generate
This tutorial are parts of the blog written here:
- https://dev.to/ashokan/sealed-secrets-bring-your-own-keys-and-multi-cluster-scenario-1ee8
#### Create acme keys
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

#### Create the application
```bash
export KUBE_CONTEXT="kubernetes-admin@ovh-krysantem-gra1-prod"

kubectl apply -f sealed-secrets/applicationset.yml -n argocd --context $KUBE_CONTEXT
```

#### Encrypt a secret
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
