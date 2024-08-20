# Run Backstage on Kubernetes

## Build the container image

1. In your application directory, build your application
```bash
    yarn install --frozen-lockfile
    yarn tsc
    yarn build:backend
```
2. Build the image
```bash
    docker image build . -f packages/backend/Dockerfile --tag backstage:1.0.0
```
## Upload the image to your container registry 
You can upload the image to any your registry, in this case, Azure Container Register (ACR)
1. Login to ACR server
```bash
    docker login ntbackstage.azurecr.io
```
2. Tag backstage:1.0.0
```bash
    docker push ntbackstage.azurecr.io/backstage:1.0.0
```
3. Push the image to <i>ntbackstage</i> container registry
```bash
    docker push ntbackstage.azurecr.io/backstage:1.0.0
```
## Deploy Postgres 
In this case we use a Azure Kubernetes Services (AKS) to create infrastructure cluster. This cluster will be used to install Backstage, ArgoCD and Crossplane
1. Connect to the Azure Kubernetes Cluster
Here, use Azure CLI
```bash
    az login
```
2. Retrieves the access credentials for a managed Kubernetes cluster from Azure and updates local kubeconfig file
```bash
    az aks get-credentials --resource-group platform-engineering --name infrastructure-cluster
```
3. Check if we can connect to the <i>infrastructure-cluster</i> cluster
```bash
    kubectl get nodes
```
4. Create the backstage namespace
```bash
    kubectl create ns backstage
```
5. Apply the manifests in the directory from this repo [portgres-resources](https://github.com/nashtech-garage/how-to-devops/tree/main/backstage/postgres-resources). It will create the Postgres resources required with a default username-password.
```bash
    kubectl apply -f postgres-resources -n backstage
```
6. Veriry the access to Postgress
```bash
    export PG_POD=$(kubectl get pods -n backstage -o=jsonpath='{.items[0].metadata.name}')
```
```bash
    kubectl exec -it --namespace=backstage $PG_POD -- /bin/bash

    bash-5.1# psql -U $POSTGRES_USER
    psql (13.2)
    backstage=# \q
    bash-5.1# exit
```
## Deploy Backstage on Kubernetes
1. Create the Backstage resources by preparing the files the apply them to the target cluster. You can find the instructions for it in the docs website as well - [creating-the-backstage-instance](https://backstage.io/docs/deployment/k8s#creating-the-backstage-instance)
2. Edit [backstage-resources/bs-secret.yaml] () with your github api token. token must have the permissions explained [here](https://backstage.io/docs/integrations/github/locations/#token-scopes).