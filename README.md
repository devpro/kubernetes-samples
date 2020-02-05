# Kubernetes samples

Samples of Kubernetes definition files.

## Requirements

You must have access to a running Kubernetes instance and have `kubectl` and `helm` commands.

```bash
# display Kubernetes server and client version
kubectl version

# list all nodes
kubectl get nodes

# display Helm version (> 3)
helm version
```

## Usecases

- [NGINX Ingress, cert-manager, Let's Encrypt](ingress-certmanager-letsencrypt/README.md)
