# Introduction
![](../../../assets/Screenshot%202024-10-08%20130453.png)

[Guide  Backstage Argo CD Plugin](https://roadie.io/backstage/plugins/argo-cd/?utm_source=backstage.io&utm_medium=marketplace&utm_campaign=argo-cd)

# Installation steps
## Install the plugin into Backstage.
```bash
cd packages/app
yarn add @roadiehq/backstage-plugin-argo-cd
```
## Add proxy config to the app-config.yaml file
```yaml
proxy:
  endpoints:
    '/argocd/api':
      target: ${ARGOCD_URL}
      changeOrigin: true
      # only if your argocd api has self-signed cert
      secure: false
      headers:
        Cookie:
          $env: ARGOCD_AUTH_TOKEN
```
##  Add argoCD widget to your overview page
```tsx
// packages/app/src/components/catalog/EntityPage.tsx
import {
  EntityArgoCDOverviewCard,
  isArgocdAvailable
} from '@roadiehq/backstage-plugin-argo-cd';

const overviewContent = (
  <Grid container spacing={3} alignItems="stretch">
  ...
    <EntitySwitch>
      <EntitySwitch.Case if={e => Boolean(isArgocdAvailable(e))}>
        <Grid item sm={4}>
          <EntityArgoCDOverviewCard />
        </Grid>
      </EntitySwitch.Case>
    </EntitySwitch>
  ...
  </Grid>
);
```
## Add annotation to the yaml config file of a component
```yaml
metadata:
  annotations:
    argocd/app-name: <your-app-name>
```
# Generate ArgoCD Auth Token
## Login to ArgoCD
```bash
argocd login <ARGOCD_SERVER> --username <ARGOCD_USER> --password <ARGOCD_PASS> --insecure --grpc-web 
```
## Edit the ArgoCD Configuration
Open the ArgoCD argocd-cm ConfigMap using the following command:
```bash
 kubectl edit configmap argocd-cm -n argocd
```
## Add API Key Capability
In the argocd-cm, look for the accounts.admin section. If it doesn't exist, you'll need to add it. Add the following line to give the admin account the apiKey capability:
```bash
 data:
  accounts.admin: apiKey
```
## Restart ArgoCD Server
To apply the changes, restart the ArgoCD server. Run the following command:
```bash
kubectl rollout restart deployment argocd-server -n argocd
```
## Generate the Token Again
```bash
argocd account generate-token --account admin
```
## Get and provide ARGOCD_AUTH_TOKEN as env variable in following format
ARGOCD_AUTH_TOKEN='argocd.token=<token>'

# Define variables in .env
export ARGOCD_AUTH_TOKEN='argocd.token=<toke>'
export ARGOCD_URL=<argo_url> (ARGOCD_URL=https://52.228.229.75/api/v1/)