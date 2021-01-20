# Flux Home

The repository for the flux configuration of my home kubernetes cluster. 

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
