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
microk8s enable dns storage ingress helm3 metrics-server
```

6. Install helm
```bash 
sudo snap install helm
```

7. Download kubectl config
```bash
microk8s config > ~/.kube/config ; chmod 600 ~/.kube/config
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

### Add sealed secrets operator
_These steps assume you have already configured your .kube/config file to allow access to the cluster via the kubectl CLI._

0. Add sealed secrets repo
```bash
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
```
1. Install sealed secrets 
```bash
helm install sealed-secrets -n kube-system sealed-secrets/sealed-secrets
```

### Enable gitops integration via argocd
_These steps assume you have already configured your .kube/config file to allow access to the cluster via the kubectl CLI._

1. Install argocd
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

2. Expose UI via a service
```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

3. Obtain the admin password to login to Argo CD portal
```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath='{.data.password}' | base64 --decode ; echo
```

4. Login to portal

## How to
### How to generate a new sealed secret

1. Create the insecure secret file or using the template provided:
```bash
kubectl -n default create secret generic example-secret \
--from-literal=key1=value1 \
--from-literal=key2=value2 \
--dry-run=client \
-o yaml > example-secret.yaml
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
