# argocd-krysantem

ArgoCD configuration files for krysantem

## Requirements
Initial cluster configuration is required, follow [our documentation](./documentation/argocd.md)

### Github variables

Create `KUBE_CONTEXT_ADMIN_$CLUSTER_NAME` in base64 running this command:
```
cat $HOME/.kube/config | base64
```

## Deploy a service
```bash
export KUBE_CONTEXT="kubernetes-admin@ovh-krysantem-gra1-prod"

kubectl apply -f cert-manager/applicationset.yml -n argocd --context $KUBE_CONTEXT
```

## Misc
### Cluster creation

Take a look at [our documentation](./documentation/k0s.md)
