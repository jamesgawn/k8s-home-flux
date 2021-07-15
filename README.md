# Flux Home

The repository for the flux configuration of my home kubernetes cluster.

## Getting Started

### Bootstrap cluster on Raspberry PIs

1. Login to your Raspberry PI
```bash
sudo sed -i '$ s/$/ cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1 swapaccount=1/' /boot/firmware/cmdline.txt
```

2. Install Microk8s
```bash
sudo snap install microk8s --classic
```

3. Update permissions to be able to execute commands without sudo
```bash
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
su - $USER
```

4. Wait until the cluster is up
```bash
microk8s status --wait-ready
```

5. Enable useful services 
```bash
microk8s enable dns storage ingress helm3
```

### _Optional:_ Adding additional nodes

1. Login to your first Raspberry PI

2. Run command to obtain join token
```bash
microk8s add-node 
```

3. Login to second Raspberry PI
   
4. Run command collected from output step 2 

5. Fin

### Enable gitops integration via flux
_These steps assume you have already configured your .kube/config file to allow access to the cluster via the kubectl CLI._
0. Install flux CLI
```bash 
curl -s https://toolkit.fluxcd.io/install.sh | sudo 
```
1. Set github personal access token (with all repo permissions):
```bash
export GITHUB_TOKEN=INSERT_TOKEN_HERE
```
2. Configure cluster
```bash
flux bootstrap github --personal --repository=k8s-home --owner=jamesgawn --read-write-key --personal --components-extra=image-reflector-controller,image-automation-controller
```

## How to
### How to generate a new sealed secret

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
kubeseal --format=yaml --cert=sealed-secret-public-cert.pem < example-secret.yaml > example-secret-sealed.yaml
```

You can obtain the public key with the following command:
```bash
kubeseal --fetch-cert \
--controller-name=sealed-secrets \
--controller-namespace=flux-system \
> sealed-secret-public-cert.pem
```
