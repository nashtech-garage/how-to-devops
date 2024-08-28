
# Crossplane to create a GKE cluster
## Install the Up command-line
1. Download and install the Upbound up command-line.
You can download the binary directly using curl or wget. Here's the command to download the correct binary
```bash
 curl -sL "https://cli.upbound.io" | sh
 mv up /usr/local/bin/
```
2. Verify the version of up with up --version
```bash
 up --version
```
##  Install Universal Crossplane
1. Install Upbound Universal Crossplane with the Up command-line.
```bash
 up uxp install
 UXP 1.16.0-up.1 installed
```
2. Verify the UXP pods are running with kubectl get pods -n upbound-system
```bash
kubectl get pods -n upbound-system
 NAME                                       READY   STATUS    RESTARTS   AGE
crossplane-77ff754998-k76zz                1/1     Running   0          40s
crossplane-rbac-manager-79b8bdd6d8-79577   1/1     Running   0          40s
```


