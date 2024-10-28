# Introduction
![](../../assets/Screenshot%202024-10-24%20172346.png)
# Provision AKS cluster with Azure DevOps as host
1. Click on "Create" button on left menu in Backstage website

![](../../assets/Screenshot%202024-10-08%20200311.png)

2. Choose "New AKS Cluster" template

![](../../assets/Screenshot%202024-10-08%20200554.png)

3. Input information about Repo location

![](../../assets/Screenshot%202024-10-08%20200920.png)

4. Basic AKS Cluster Configuration

![](../../assets/Screenshot%202024-10-08%20201151.png)

5. Review and Create Cluster

![](../../assets/Screenshot%202024-10-08%20201252.png)

![](../../assets/Screenshot%202024-10-08%20201440.png)

6. After creating cluster, a component will be created

![](../../assets/Screenshot%202024-10-08%20202719.png)


![](../../assets/Screenshot%202024-10-08%20202800.png)

## Verify 
1. Verify a new repository name 'my-repo-1' created on Azure DevOps in Organization 'lecao' and Project 'back-stack'

![](../../assets/Screenshot%202024-10-08%20201713.png)

This is the repo that GitHub action used to provision AKS cluster

2. Verify action in GitHub action

![](../../assets/Screenshot%202024-10-08%20201913.png)

![](../../assets/Screenshot%202024-10-08%20202000.png)

This action will call ArgoCD to create ArgoCD applications and provision AKS cluster

Or you can view action in Backstage Catalog

![](../../assets/Screenshot%202024-10-08%20222805.png)

![](../../assets/Screenshot%202024-10-08%20222841.png)

![](../../assets/Screenshot%202024-10-08%20222922.png)

3. Verify applications created in ArgoCD

![](../../assets/Screenshot%202024-10-08%20202146.png)

![](../../assets/Screenshot%202024-10-08%20202222.png)

![](../../assets/Screenshot%202024-10-08%20202346.png)

4. Verify AKS cluster created in Azure 

![](../../assets/Screenshot%202024-10-08%20202453.png)

# Provision AKS cluster with github.com as host
The flow is the same with Provision AKS cluster with Azure DevOps as host. The differce is source code is hosted on github.com

![](../../assets/Screenshot%202024-10-08%20223250.png)