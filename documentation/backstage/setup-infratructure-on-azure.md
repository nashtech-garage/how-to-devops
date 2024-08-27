# Pre-requisite 
- A AKS cluster setup 
# Connect to the Azure Kubernetes Cluster
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

# Creating a namespace
Deployments in Kubernetes are commonly assigned to their own namespace to isolate services in a multi-tenant environment.
This can be done through kubectl directly:
```bash
    kubectl create namespace backstage
    namespace/backstage created
```
Alternatively, create and apply a Namespace definition:
```bash
# k8s-namespace.yaml
    apiVersion: v1
    kind: Namespace
    metadata:
    name: backstage
```
```bash
    kubectl apply -f kubernetes/namespace.yaml
    namespace/backstage created
```
# Creating the PostgreSQL database
Backstage in production uses PostgreSQL as a database. To isolate the database from Backstage app deployments, we can create a separate Kubernetes deployment for PostgreSQL.
## Creating a PostgreSQL secret
First, create a Kubernetes Secret for the PostgreSQL username and password. This will be used by both the PostgreSQL database and Backstage deployments:
```bash
# postgres-resources/pg-secret.yaml
# kubernetes/pg-secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: postgres-secrets
  namespace: backstage
type: Opaque
data:
  POSTGRES_HOST: cG9zdGdyZXM=
  POSTGRES_USER: YmFja3N0YWdl
  POSTGRES_PASSWORD: aHVudGVyMg==
  POSTGRES_DB: cHJvZHVjdGlvbl9iYWNrc3RhZ2U=
```
The data in Kubernetes secrets are base64-encoded. The values can be generated on the command line:
```bash
$ echo -n "backstage" | base64
YmFja3N0YWdl
```
> Secrets are base64-encoded, but not encrypted. Be sure to enable Encryption at Rest for the cluster. For storing secrets in Git, consider SealedSecrets or other solutions.
## Creating a PostgreSQL persistent volume
PostgreSQL needs a persistent volume to store data; we'll create one along with a PersistentVolumeClaim. In this case, we're claiming the whole volume - but claims can ask for only part of a volume as well.
```bash
# postgres-resources/pg-vol.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: postgres-storage
  namespace: backstage
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 2G
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: '/mnt/data'
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-storage-claim
  namespace: backstage
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2G
```
## Creating a PostgreSQL deployment
Now we can create a Kubernetes Deployment descriptor for the PostgreSQL database deployment itself:
```bash
# postgres-resources/pg-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: backstage
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:13.2-alpine
          imagePullPolicy: 'IfNotPresent'
          ports:
            - containerPort: 5432
          envFrom:
            - secretRef:
                name: postgres-secrets
          volumeMounts:
            - mountPath: /var/lib/postgresql/data
              name: postgresdb
      volumes:
        - name: postgresdb
          persistentVolumeClaim:
            claimName: postgres-storage-claim
```
## Creating a PostgreSQL service
Kubernetes pods are transient - they can be stopped, restarted, or created dynamically. Therefore we don't want to try to connect to pods directly, but rather create a Kubernetes Service. Services keep track of pods and direct traffic to the right place.

The final step for our database is to create the service descriptor:
```bash
# postgres-resources/pg-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: backstage
spec:
  selector:
    app: postgres
  ports:
    - port: 5432
```
## Apply all portgres manifests
> Apply the manifests in the directory from this repo [portgres-resources](https://github.com/nashtech-garage/how-to-devops/tree/main/backstage/postgres-resources). It will create the Postgres resources required with a default username-password.
```bash
    kubectl apply -f postgres-resources -n backstage
```
 Veriry the access to Postgress
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
# Creating the Backstage instance
Now that we have PostgreSQL up and ready to store data, we can create the Backstage instance. This follows similar steps as the PostgreSQL deployment.
## Creating a Backstage secret
```bash
# backstage-resources/bs-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: backstage-secrets
  namespace: backstage
type: Opaque
data:
  APP_BASEURL: aHR0cDovL2xvY2FsaG9zdDo4MDgw
  BACKEND_BASEURL: aHR0cDovL2xvY2FsaG9zdDo4MDgw
  BACKEND_PORT: NzAwNw==
```
## Creating a ACR secret
Create dockerconfigjson file
```bash
 # dockercongig.json
{
    "auths": {
      "ntbackstage.azurecr.io": {
        "username": <Your ntbackstage username>
        "password": <Your ntbackstage password>
      }
    }
  }
  
```
```bash
# backstage-resources/acr-secret.yaml

apiVersion: v1
kind: Secret
metadata:
  name: acr-secret
  namespace: backstage
data:
  .dockerconfigjson: <youre docker config base64>
type: kubernetes.io/dockerconfigjson

```
## Creating a auth secret

```bash
# backstage-resources/bs-auth-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: auth-secrets
  namespace: backstage
type: Opaque
data:
  GITHUB_CLIENT_ID: T3YyM2xpUHloQmgwZVRucllOTXM=
  GITHUB_CLIENT_SECRET: MDYwOWQ4MzY2NzczZDliYTU2NzhlN2RlYWNiYTI0MDZjMDUxYTRhOA==
  AZURE_CLIENT_ID: NTJmNDU5MDYtNTVjYy00ZTc4LTkzMTctNDczY2Q2ZTE1NDc1
  AZURE_CLIENT_SECRET: OXFyOFF+bDEyV1pySHBRLU9kaHlkS2pNUzY3U3B1QmNaamZQWGRmZA==
  AZURE_TENANT_ID: Y2NiYmNkMjMtNzQ3NS00NjVkLTkyM2UtZDZmZGQzNDJmMjA5

```
## Creating a auth secret

```bash
# backstage-resources/bs-integration-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: integration-secrets
  namespace: backstage
type: Opaque
data:
  GITHUB_TOKEN: Z2hwX1o0RUFrT0tDYWplTklqMlZWbDQzWGVwaFRZZVRJbjBWQjVhNw==

```

## Creating a Backstage deployment
```bash
# backstage-resources/bs-deploy.yaml

# kubernetes/backstage.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backstage
  namespace: backstage
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backstage
  template:
    metadata:
      labels:
        app: backstage
    spec:
      serviceAccountName: backstage-service-account
      containers:
        - name: backstage
          image: ntbackstage.azurecr.io/backstage:1.0.0
          imagePullPolicy: Always
          ports:
            - name: http
              containerPort: 7007
          envFrom:
            - secretRef:
                name: backstage-secrets
            - secretRef:
                name: postgres-secrets
            - secretRef:
                name: integration-secrets
            - secretRef:
                name: auth-secrets
      imagePullSecrets:
        - name: acr-secret  # Add the secret created for pulling images from ACR
# Uncomment if health checks are enabled in your app:
# https://backstage.io/docs/plugins/observability#health-checks
#          readinessProbe:
#            httpGet:
#              port: 7007
#              path: /healthcheck
#          livenessProbe:
#            httpGet:
#              port: 7007
#              path: /healthcheck
```
## Creating a Backstage service
```bash
# backstage-resources/bs-svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: backstage-service
  namespace: backstage
spec:
  type: ClusterIP
  selector:
    app: backstage
  ports:
    - name: http
      port: 80
      targetPort: 7007


```
## Set Up the Ingress Resource
### Install NGINX Ingress Controller
If you haven't installed an Ingress Controller (like NGINX Ingress Controller) on your AKS cluster, you need to do that first.
Fowllowing [link](https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx)
### Create an Ingress Resource for Backstage
```bash
# backstage-resources/bs-ingress.yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: backstage-ingress
  namespace: backstage
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: backstage.nashtech.platformengineering.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: backstage-service
            port:
              number: 80
```
## Apply all portgres manifests
> Apply the manifests in the directory from this repo [backstage-resources](https://github.com/nashtech-garage/how-to-devops/tree/main/backstage/backstage-resources). It will create the backstage-resources.
```bash
      kubectl apply -f backstage-resources -n backstage
```