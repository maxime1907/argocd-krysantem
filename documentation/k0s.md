# Cluster installation

Following this guide will install a single node cluster with:
- calico to handle network
- containerd as the container engine
- openebs for local storage CSI

## Install
```bash
curl -sSLf https://get.k0s.sh | sudo sh
```

## Generate k0s config file with default storage provider
- https://docs.k0sproject.io/v1.23.8+k0s.0/storage/
- https://docs.k0sproject.io/v1.23.8+k0s.0/configuration/

## Install k0s cluster
```bash
sudo k0s install controller --single
sudo k0s start
```

## Wait for node to come up online
```bash
watch -n 5 sudo k0s kubectl get nodes
```

## Get cluster admin kubeconfig
```bash
sudo k0s kubeconfig admin
```

## Configure remote access on your desktop
```bash
mkdir -p ~/.kube
```
## Paste the output of cluster admin kubeconfig inside
```bash
nano ~/.kube/config
```
