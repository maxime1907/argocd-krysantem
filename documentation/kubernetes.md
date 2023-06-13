# Recover from network failure

```bash
systemctl stop kubelet
nerdctl stop $(nerdctl ps -a -q)
nerdctl rm $(nerdctl ps -a -q)
nerdctl system prune
systemctl restart containerd
iptables -F
systemctl restart etcd
systemctl restart kubelet
```

# Delete ingress-nginx load balancer

```bash
kubectl delete ApplicationSet ingress-nginx -n argocd
```
