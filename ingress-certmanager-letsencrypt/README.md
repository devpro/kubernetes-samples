# NGINX Ingress, cert-manager, Let's Encrypt

## Requirements

### Azure

- Create a new DNS zone (poc.automation.kirrk.com)
  - Add a A record with the IP address (no prefix)

## Setup

```bash
# install nginx
helm install nginxingress stable/nginx-ingress --namespace default

# get the ip (wait for it to be created)
kubectl get svc nginxingress-nginx-ingress-controller -o jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace default

# opening the ip in a browser should send you back "default backend - 404" as nothing is defined

# question: add the IP as a A entry in the DNS?
az network dns record-set a add-record --resource-group myResourceGroup --zone-name MY_CUSTOM_DOMAIN --record-set-name * --ipv4-address MY_EXTERNAL_IP

# install with helm a demo application (here Joomla, a CMS, see https://github.com/helm/charts/tree/master/stable/joomla)
helm install joomlasample stable/joomla --set joomlaPassword=secretpassword --set mariadb.root.password=secretpassword --set service.type=ClusterIP --set ingress.enabled=true --set ingress.hosts[0].name=joomla.poc.automation.kirrk.com

# I had to force the domain in my host file
# opening http://joomla.poc.automation.kirrk.com/ works but with a self signed certificate

# create the namespaces (see https://cert-manager.io/docs/installation/kubernetes/)
helm repo add jetstack https://charts.jetstack.io
helm repo update
kubectl apply --validate=false -f https://raw.githubusercontent.com/jetstack/cert-manager/v0.13.0/deploy/manifests/00-crds.yaml --namespace default

# create a cluster issuer
kubectl apply -f letsencrypt-poc.yaml --namespace default

# install cert-manager
helm install cert-manager --namespace default --version v0.13.0 jetstack/cert-manager --set ingressShim.defaultIssuerName=letsencrypt-poc --set ingressShim.defaultIssuerKind=ClusterIssuer

# check the installation of cert-manager
kubectl get pods --namespace default

helm upgrade joomlasample stable/joomla --set joomlaPassword=secretpassword --set mariadb.root.password=secretpassword --set service.type=ClusterIP --set ingress.enabled=true --set ingress.certManager=true --set ingress.tls[0].secretName=joomla.local-tls --set ingress.hosts[0].name=joomla.poc.automation.kirrk.com

helm repo add azure-samples https://azure-samples.github.io/helm-charts/

helm install aks-helloworld azure-samples/aks-helloworld --namespace default

helm install aks-helloworld-two azure-samples/aks-helloworld --namespace default --set title="AKS Ingress Demo" --set serviceName="aks-helloworld-two"

kubectl apply -f hello-world-ingress.yaml --namespace default

# See https://cert-manager.io/docs/faq/acme/
kubectl get certificate --namespace default

kubectl describe certificate tls-secret

kubectl describe CertificateRequest

kubectl describe order tls-secret-1856864338-4193228896

kubectl describe ingress hello-world-ingress

kubectl get challenges
```

## References

- [Create an HTTPS ingress controller on Azure Kubernetes Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/ingress-tls)
- [Secure Kubernetes Services With Ingress, TLS And Let's Encrypt](https://docs.bitnami.com/kubernetes/how-to/secure-kubernetes-services-with-ingress-tls-letsencrypt/)
