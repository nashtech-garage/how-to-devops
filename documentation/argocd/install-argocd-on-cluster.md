# Pre-requisite 
- A cluster setup- here is using AKS cluster
# Install Argo CD on a cluster
## Create a namespace for ArgoCD
```bash
 kubectl create namespace argocd
```
## Run the Argo CD install script provided by the project maintainers
```bash
 kubectl apply -n argocd -f  https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
> Notes: By default ArgoCD is not publicly assessable so we will make some changed to the argo-server in order to access the ArgoCD user interface via Port-Forwarding or Load Balancer.

## Access the ArgoCD user Load Balancer 
Patching ArgoCD Server to use a load balancer and Obtain the Server IP and Password
```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```
## Retrieve the API server public IP address and the server password 
```bash
export ARGOCD_SERVER=`kubectl get svc argocd-server -n argocd -o json | jq --raw-output '.status.loadBalancer.ingress[0].ip'`
echo "ARGOCD_SERVER IP: $ARGOCD_SERVER"

export ARGOCD_PASSWORD=`kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d`
echo "ARGOCD_SERVER PASSWORD: $ARGOCD_PASSWORD"
```
> Notes: The command above might require you to install jq, a versatile tool used for manipulating and parsing JSON data.

## Access the ArgoCD user interface (UI)
Now, utilizing the ArgoCD server IP and the server password, we can access the ArgoCD user interface (UI). When accessing the server using the provided IP address, you will be prompted to log in. The default username for ArgoCD is "admin." After a successful login, you will be able to access and interact with the ArgoCD user interface
![](./assets/Screenshot%202024-08-26%20225744.png)

## Install Argo CD CLI
```bash
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64

```
## Login to Argo CD using CLI
```bash
argocd login <Ingress IP> --username <username> --password <password> --insecure

```
# Create new user (Optional)
New users should be defined in argocd-cm ConfigMap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
  labels:
    app.kubernetes.io/name: argocd-cm
    app.kubernetes.io/part-of: argocd
data:
  # add an additional local user with apiKey and login capabilities
  #   apiKey - allows generating API keys
  #   login - allows to login using UI
  accounts.lecao: apiKey, login
  # disables user. User is enabled by default
  accounts.lecao.enabled: "true"
```