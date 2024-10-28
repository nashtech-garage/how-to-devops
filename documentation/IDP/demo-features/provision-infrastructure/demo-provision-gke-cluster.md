# Provision GKE cluster with GitHub as host
1. Click on "Create" button on left menu in Backstage website

![](../../assets/Screenshot%202024-10-08%20200311.png)

2. Choose "New GKE Cluster" template

![](../../assets/Screenshot%202024-10-28%20230511.png)

3. Input information about Repo location

![](../../assets/Screenshot%202024-10-28%20230601.png)

4. Basic GKR Cluster Configuration

![](../../assets/Screenshot%202024-10-28%20230737.png)

5. Review and Create Cluster

![](../../assets/Screenshot%202024-10-28%20230810.png)

![](../../assets/Screenshot%202024-10-28%20230850.png)

6. After creating cluster, a component will be created

![](../../assets/Screenshot%202024-10-28%20222038.png)


![](../../assets/Screenshot%202024-10-28%20231039.png)

## Verify 
1. Verify a new repository name 'my-gke-repo-1' created in GitHub with Owner is hoanglecao

![](../../assets/Screenshot%202024-10-28%20231128.png)

This is the repo that GitHub action used to provision GKE cluster

2. Verify action in GitHub action

![](../../assets/Screenshot%202024-10-28%20231222.png)


3. Verify applications created in ArgoCD

![](../../assets/Screenshot%202024-10-28%20231429.png)

![](../../assets/Screenshot%202024-10-28%20231507.png)

![](../../assets/Screenshot%202024-10-28%20231530.png)

4. Verify GKE cluster created in Google Cloud 

![](../../assets/Screenshot%202024-10-28%20231719.png)

