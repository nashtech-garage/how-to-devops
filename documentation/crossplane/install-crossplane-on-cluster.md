# Pre-requisite 
- A cluster setup- here is using AKS cluster
# Install Argo CD on a cluster
## Create a namespace for Crossplane
```bash
 kubectl create namespace crossplane
```
## Add the Crossplane Helm repository
```bash
 helm repo add crossplane-stable https://charts.crossplane.io/stable
 helm repo update
```
##  Install Crossplane
Install Crossplane using Helm
```bash
 helm install crossplane --namespace crossplane --version <crossplane-app-version> crossplane-stable/crossplane
```

> Notes: Get crossplane-app-version by running command
```bash
    helm search repo crossplane
```


