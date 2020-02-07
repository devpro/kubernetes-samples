# NGINX Ingress, cert-manager, Let's Encrypt

## Requirements

### Azure

- Create a new DNS zone and register the Azure DNS name servers in the system that is the owner of the domain (OVH or 1&1 for example)

## Steps

### Install

```bash
# /!\ make sure you are on the right namespace

# install nginx
helm install nginx stable/nginx-ingress

# you can have a look at the configmap that is created, containing NGINX ingress configuration
kubectl describe configmap nginxingress-nginx-ingress-controller

# get the ip (wait for it to be created)
kubectl get svc nginx-nginx-ingress-controller -o jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace mynamespace

# opening the ip in a browser should send you back "default backend - 404" as nothing is defined

# Azure: add the IP as a A entry in the DNS
az network dns record-set a add-record --resource-group myResourceGroup --zone-name MY_CUSTOM_DOMAIN --record-set-name * --ipv4-address MY_EXTERNAL_IP

# create the namespaces (see https://cert-manager.io/docs/installation/kubernetes/)
helm repo add jetstack https://charts.jetstack.io
helm repo update
kubectl apply --validate=false -f https://raw.githubusercontent.com/jetstack/cert-manager/v0.13.0/deploy/manifests/00-crds.yaml --namespace mynamespace

# create a cluster issuer
kubectl apply -f letsencrypt-poc.yaml --namespace mynamespace

# install cert-manager
helm install cert-manager --namespace mynamespace --version v0.13.0 jetstack/cert-manager --set ingressShim.defaultIssuerName=letsencrypt-poc --set ingressShim.defaultIssuerKind=ClusterIssuer

# check the installation of cert-manager
kubectl get pods --namespace mynamespace

# use samples to have a try
helm repo add azure-samples https://azure-samples.github.io/helm-charts/

helm install aks-helloworld azure-samples/aks-helloworld --namespace mynamespace

helm install aks-helloworld-two azure-samples/aks-helloworld --namespace mynamespace --set title="AKS Ingress Demo" --set serviceName="aks-helloworld-two"

kubectl apply -f hello-world-ingress.yaml --namespace mynamespace

kubectl get certificate --namespace mynamespace

# if the DNS has been correctly configured, then everything should be fine :) and by opening the url you have a valid certificate
```

### Investigate

```bash
# See https://cert-manager.io/docs/faq/acme/

kubectl describe certificate tls-secret

kubectl describe CertificateRequest

kubectl describe order tls-secret-1856864338-4193228896

kubectl describe ingress hello-world-ingress

kubectl get challenges
```

## References

- [Create an HTTPS ingress controller on Azure Kubernetes Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/ingress-tls)
- [Secure Kubernetes Services With Ingress, TLS And Let's Encrypt](https://docs.bitnami.com/kubernetes/how-to/secure-kubernetes-services-with-ingress-tls-letsencrypt/)
