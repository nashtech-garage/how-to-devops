# Introduction
## Backstage Software Templates
The Software Templates part of Backstage is a tool that can help you create Components inside Backstage. By default, it has the ability to load skeletons of code, template in some variables, and then publish the template to some locations like GitHub or GitLab or Azure DevOps
## Create AKS template
Create template.yaml in packages/templates/aks-cluster
```yaml
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: aks-cluster
  title: New AKS Cluster
  description: An  template for the scaffolder that provision a AKS cluster using Crossplane and ArgoCD
spec:
  owner: user:le.caothihoang
  type: infrastructure

  parameters:  
    - title: Choose a Repo location
      required:
        - repoUrl
      properties:
        repoUrl:
          title: Repository Location for ArgoCD to Deploy
          type: string
          ui:field: RepoUrlPicker
          ui:options:
            allowedHosts:
              - github.com
              - dev.azure.com
    - title: Basic AKS Cluster Configuration
      required:
        - clusterName
        - region
        - resourceGroupName
        - kubernetesVersion
        - dnsPrefix
      properties:
        clusterName:
          title: AKS Cluster Name
          type: string
          description: The name of the AKS cluster to create
          ui:autofocus: true
        region:
          title: Region
          type: string
          description: The region where the AKS cluster will be deployed
          enum:
            - eastus
            - eastus2
            - centralus
            - northeurope
            - westeurope
            - europe-west2
            - eastasia
            - southeastasia
        resourceGroupName:
          type: string
          title: Resource Group Name
          description: Name of the Azure Resource Group to be used or created
        kubernetesVersion:
          title: Kubernetes version
          type: string
          description: Kubernetes version of the Cluster
          default: "1.29.7"
          enum:
            - "1.29.7"
            - "1.29.6"
            - "1.29.5"
            - "1.29.4"
            - "1.29.3"
        dnsPrefix:
          type: string
          title: DNS Prefix
          description: The DNS prefix for the AKS cluster
  steps:
    - id: fetch-base-aks-skeleton
      name: Fetch AKS Cluster skeleton
      action: fetch:template
      input:
        url: 'https://github.com/nashtech-garage/crossplane-aks-skeleton'
        targetPath: .
        values:
          clusterName: ${{ parameters.clusterName }}
          region: ${{ parameters.region }}
          resourceGroupName: ${{ parameters.resourceGroupName }}
          kubernetesVersion: ${{ parameters.kubernetesVersion}}
          dnsPrefix: ${{ parameters.dnsPrefix }}        
          owner: ${{ parameters.repoUrl | parseRepoUrl | pick('owner')}} 
          repo: ${{ parameters.repoUrl | parseRepoUrl | pick('repo')}} 
    # This step publishes to Azure DevOps
    - id: publish
      name: Publish to Github
      if: ${{ parameters.repoUrl | parseRepoUrl | pick('host') === "github.com" }}
      action: publish:github
      input:
        allowedHosts: ['github.com']
        description: This is ${{ parameters.clusterName }}
        repoUrl: ${{ parameters.repoUrl }}
        # This step publishes to Azure DevOps
    - id: publish
      name: Publish to Azure DevOps
      if: ${{ parameters.repoUrl | parseRepoUrl | pick('host') === "dev.azure.com" }}
      action: publish:azure
      input:
        allowedHosts: ['dev.azure.com']
        description: This is ${{ parameters.clusterName }}
        repoUrl: ${{ parameters.repoUrl }}
        organization: ${{ parameters.repoUrl | parseRepoUrl | pick('organization') }}
        project: ${{ parameters.repoUrl | parseRepoUrl | pick('project') }}
        repo: ${{ parameters.repoUrl | parseRepoUrl | pick('repo') }}
    # Start a GitHub Action to build an GKE cluster with Crossplane
    - id: github-action
      name: Trigger GitHub Action
      action: github:actions:dispatch
      input:
        workflowId: deploy_aks_with_argocd.yaml
        repoUrl: 'github.com?owner=nashtech-garage&repo=how-to-devops'
        branchOrTagName: 'main'
        workflowInputs:
          clusterName: ${{ parameters.clusterName }}
          region: ${{ parameters.region }}
          resourceGroupName: ${{ parameters.resourceGroupName }}
          kubernetesVersion: ${{ parameters.kubernetesVersion}}
          dnsPrefix: ${{ parameters.dnsPrefix }}          
          repoURLforArgo: ${{ steps['publish'].output.remoteUrl }} 
          host: ${{ parameters.repoUrl | parseRepoUrl | pick('host') }}

     # The final step is to register our new component in the catalog.
    - id: register
      name: Register
      action: catalog:register
      input:
        repoContentsUrl: ${{ steps['publish'].output.repoContentsUrl }}
        catalogInfoPath: '/catalog-info.yaml'
  output:
    links:
      - title: A new repository created
        url: ${{ steps['publish'].output.remoteUrl }}
```
## Add the template files to the catalog
Add below in app-config.yaml

```yaml
# deploy_aks_with_argocd.yaml
catalog:
  locations:
   # aks template
    - type: file
      target: ../../packages/backend/templates/aks-cluster/template.yaml
      rules:
        - allow: [Template]
```
## Create workflow in github action
repoUrl: 'github.com?owner=nashtech-garage&repo=how-to-devops'

```yaml
name: Manage AKS Cluster

on:
  workflow_dispatch:
    inputs:
      clusterName:
        description: 'Cluster Name'
        required: true
      region:
        description: 'Region'
        required: true
      resourceGroupName:
        description: 'Resource Group Name'
        required: true
      kubernetesVersion:
        description: 'Kubernetes Version'
        required: true
      dnsPrefix:
        description: 'DNS Prefix'
        required: true
      repoURLforArgo: 
        description: 'The URL of the repository for Argo CD to deploy GKE'
        required: true
      host:
        description: 'github.com or azure.dev.com'
        required: true
jobs:
  provision-microsoft-dev-box:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout Code
      uses: actions/checkout@v2
    
    - name: Install jq
      run: sudo apt-get install -y jq

    - name: Install Argo CD CLI
      run: |
        sudo curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
        sudo chmod +x /usr/local/bin/argocd

    - name: Login to Argo CD 
      run: argocd login ${{ secrets.ARGOCD_SERVER }} --username ${{ secrets.ARGOCD_USER }} --password ${{ secrets.ARGOCD_PASS }} --insecure --grpc-web 

    - name: Register Repository in Argo CD
      run: |
          if [[ ${{ github.event.inputs.host }} == "github.com" ]]; then
            echo "Registering GitHub repository in ArgoCD..."
            argocd repo add ${{ github.event.inputs.repoURLforArgo }} --username hoanglecao --password ${{ secrets.MYGITHUB_TOKEN }}
          
          elif [[ ${{ github.event.inputs.host }} == "dev.azure.com" ]]; then
            echo "Registering Azure DevOps repository in ArgoCD..."
            argocd repo add ${{ github.event.inputs.repoURLforArgo }} --username lecao --password ${{ secrets.MYAZURE_TOKEN }}
          
          else
            echo "Unsupported repository host: $REPO_HOST"
            exit 1
          fi

    - name: Create ArgoCD app for Azure provider
      run: |
          # Check if the ArgoCD app exists
          if argocd app get azure-provider-app > /dev/null 2>&1; then
            echo "azure-provider-app already exists."
          else
            argocd app create azure-provider-app \
              --repo ${{ github.event.inputs.repoURLforArgo }} \
              --path azure \
              --dest-server https://kubernetes.default.svc \
              --dest-namespace azure-provider \
              --project default \
              --sync-policy automated \
              --sync-option CreateNamespace=true \
              --sync-retry-limit 5 \
              --upsert \
              --grpc-web
          fi
    - name: Wait for azure-provider-app to become synced
      run: |
        TIMEOUT=660
        INTERVAL=30
        ELAPSED=0

        while [[ $ELAPSED -lt $TIMEOUT ]]; do
          APP_STATUS=$(argocd app get azure-provider-app --output json)
          HEALTH_STATUS=$(echo "$APP_STATUS" | jq -r '.status.health.status')
          SYNC_STATUS=$(echo "$APP_STATUS" | jq -r '.status.sync.status')

          echo "Application health status: $HEALTH_STATUS"
          echo "Application sync status: $SYNC_STATUS"

          if [[ "$HEALTH_STATUS" == "Healthy" && "$SYNC_STATUS" == "Synced" ]]; then
            echo "azure-provider-app is healthy and synced. Proceeding..."
            break
          fi

          echo "Waiting for azure-provider-app to become healthy and synced..."
          sleep $INTERVAL
          ELAPSED=$((ELAPSED + INTERVAL))
        done

        if [[ "$HEALTH_STATUS" != "Healthy" || "$SYNC_STATUS" != "Synced" ]]; then
          echo "azure-provider-app did not become healthy and synced within $TIMEOUT seconds. Exiting..."
          exit 1
        fi
    - name: Create ArgoCD app for azure provider
      run: |
          # Check if the ArgoCD app exists
          if argocd app get azure-provider-app > /dev/null 2>&1; then
            echo "azure-provider-app already exists"
          else
            argocd app create azure-provider-app  \
              --repo ${{ github.event.inputs.repoURLforArgo }} \
              --path aks-cluster \
              --dest-server https://kubernetes.default.svc \
              --dest-namespace azure-provider \
              --project default \
              --sync-policy automated \
              --sync-retry-limit 5 \
              --upsert \
              --grpc-web
          fi

    - name: Wait for azure-provider-app to become synced
      run: |
            # Initialize time
            TIMEOUT=660
            INTERVAL=30
            ELAPSED=0

            # Loop until the app becomes healthy and synced or we hit the timeout
            while [[ $ELAPSED -lt $TIMEOUT ]]; do
              # Get the health status and sync status of the application
              APP_STATUS=$(argocd app get azure-provider-app --output json)
              HEALTH_STATUS=$(echo "$APP_STATUS" | jq -r '.status.health.status')
              SYNC_STATUS=$(echo "$APP_STATUS" | jq -r '.status.sync.status')

              echo "Application health status: $HEALTH_STATUS"
              echo "Application sync status: $SYNC_STATUS"

              # If the app is both healthy and synced, break out of the loop
              if [[ "$HEALTH_STATUS" == "Healthy" && "$SYNC_STATUS" == "Synced" ]]; then
                echo "azure-provider-app is healthy and synced. Proceeding..."
                break
              fi

              # Wait for the interval duration before checking again
              echo "Waiting for azure-provider-app to become healthy and synced..."
              sleep $INTERVAL

              # Increment the elapsed time
              ELAPSED=$((ELAPSED + INTERVAL))
            done

            # If the app is still not healthy or synced after the timeout, exit with an error
            if [[ "$HEALTH_STATUS" != "Healthy" || "$SYNC_STATUS" != "Synced" ]]; then
              echo "azure-provider-app did not become healthy and synced within $TIMEOUT seconds. Exiting..."
              exit 1
            fi
    - name: Create ArgoCD app for AKS Cluster
      run: |
          argocd app create aks-${{ github.event.inputs.clusterName }}-app \
            --repo ${{ github.event.inputs.repoURLforArgo }} \
            --path aks-cluster \
            --dest-server https://kubernetes.default.svc \
            --dest-namespace azure-provider \
            --project default \
            --sync-policy automated \
            --sync-retry-limit 5 \
            --upsert \
            --grpc-web

    - name: Wait for aks-${{ github.event.inputs.clusterName }}-app to become synced
      run: |
        # Initialize time
        TIMEOUT=660
        INTERVAL=30
        ELAPSED=0
        # Loop until the app becomes healthy and synced or we hit the timeout
        while [[ $ELAPSED -lt $TIMEOUT ]]; do
          # Get the health status and sync status of the application
          APP_STATUS=$(argocd app get aks-${{ github.event.inputs.clusterName }}-app --output json)
          HEALTH_STATUS=$(echo "$APP_STATUS" | jq -r '.status.health.status')
          SYNC_STATUS=$(echo "$APP_STATUS" | jq -r '.status.sync.status')

          echo "Application health status: $HEALTH_STATUS"
          echo "Application sync status: $SYNC_STATUS"

          # If the app is both healthy and synced, break out of the loop
          if [[ "$HEALTH_STATUS" == "Healthy" && "$SYNC_STATUS" == "Synced" ]]; then
            echo "aks-${{ github.event.inputs.clusterName }}-app is healthy and synced. Proceeding..."
            break
          fi

          # Wait for the interval duration before checking again
          echo "Waiting for aks-${{ github.event.inputs.clusterName }}-app to become healthy and synced..."
          sleep $INTERVAL

          # Increment the elapsed time
          ELAPSED=$((ELAPSED + INTERVAL))
        done

        # If the app is still not healthy or synced after the timeout, exit with an error
        if [[ "$HEALTH_STATUS" != "Healthy" || "$SYNC_STATUS" != "Synced" ]]; then
          echo "aks-${{ github.event.inputs.clusterName }}-app did not become healthy and synced within $TIMEOUT seconds. Exiting..."
          exit 1
        fi

```
## Setup repository in GitHub Action
![](../assets/Screenshot%202024-10-08%20173316.png)

### Create ArgoCD secrets 
- secrets.ARGOCD_SERVER
- secrets.ARGOCD_USER
- secrets.ARGOCD_PASS

![](./assets/Screenshot%202024-10-08%20171448.png)

Refer to [Install ArgoCD on Cluster](../../argocd/install-argocd-on-cluster.md) 

### Create GitHub Token 
- secrets.MYGITHUB_TOKEN

Refer to [Create github token](../../github/create-github-token.md)

### Create Azure Token 
- secrets.MYAZURE_TOKEN

Refer to [Create azure devops token](../../azure/create-azure-devops-token.md)