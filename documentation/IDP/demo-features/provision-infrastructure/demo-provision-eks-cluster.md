# Provision EKS cluster with GitHub as host
1. Click on "Create" button on left menu in Backstage website

![](../../assets/Screenshot%202024-10-08%20200311.png)

2. Choose "New EKS Cluster" template

![](../../assets/Screenshot%202024-10-28%20221103.png)

3. Input information about Repo location

![](../../assets/Screenshot%202024-10-28%20221230.png)

4. Basic EKS Cluster Configuration

![](../../assets/Screenshot%202024-10-28%20221743.png)

5. Review and Create Cluster

![](../../assets/Screenshot%202024-10-28%20221834.png)

![](../../assets/Screenshot%202024-10-28%20221937.png)

6. After creating cluster, a component will be created

![](../../assets/Screenshot%202024-10-28%20222038.png)


![](../../assets/Screenshot%202024-10-28%20222112.png)

## Verify 
1. Verify a new repository name 'my-eks-repo-1' created in GitHub with Owner is hoanglecao

![](../../assets/Screenshot%202024-10-28%20222436.png)

This is the repo that GitHub action used to provision EKS cluster

2. Verify action in GitHub action

![](../../assets/Screenshot%202024-10-28%20222616.png)

![](../../assets/Screenshot%202024-10-28%20222655.png)

This action will call ArgoCD to create ArgoCD applications and provision EKS cluster

Or you can view action in Backstage Catalog

![](../../assets/Screenshot%202024-10-08%20222805.png)

![](../../assets/Screenshot%202024-10-08%20222841.png)

![](../../assets/Screenshot%202024-10-28%20222809.png)

3. Verify applications created in ArgoCD

![](../../assets/Screenshot%202024-10-28%20222915.png)

![](../../assets/Screenshot%202024-10-28%20222951.png)

![](../../assets/Screenshot%202024-10-28%20223033.png)

![](../../assets/Screenshot%202024-10-28%20223110.png)

4. Verify EKS cluster created in AWS 

![](../../assets/Screenshot%202024-10-28%20223246.png)

