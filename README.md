# Flux Home

The repository for the flux configuration of my home kubernetes cluster. 

## Bootstrapping Cluster
_These steps assume you have already configured your .kube/config file to allow access to the cluster via the kubectl CLI._
0. Install flux CLI
```bash 
curl -s https://toolkit.fluxcd.io/install.sh | sudo 
```
1. Set github personal access token for repository:
```bash
export GITHUB_TOKEN=INSERT_TOKEN_HERE
```
2. Configure cluster 
```bash
flux bootstrap github --personal --repository=flux-home --owner=jamesgawn --components-extra=image-reflector-controller,image-automation-controller
```
## How to generate a new sealed secret

1. Create the insecure secret file or using the template provided:
```bash
kubectl -n default create secret generic example-secret \
--from-literal=key1=value1 \
--from-literal=key2=value2 \
--dry-run \
-o yaml > basic-auth.yaml
```
```yaml
apiVersion: v1
data:
  key1: values1
  key2: values2
kind: Secret
metadata:
  name: name
  namespace: namespace

```
2. Secure it using public key 
```bash
kubeseal --format=yaml --cert=pub-sealed-secret-cert.pem < example-secret.yaml > example-secret-sealed.yaml
```

You can obtain the public key with the following command:
```bash
kubeseal --fetch-cert \
--controller-name=sealed-secrets \
--controller-namespace=flux-system \
> pub-sealed-secret-cert.pem
```
