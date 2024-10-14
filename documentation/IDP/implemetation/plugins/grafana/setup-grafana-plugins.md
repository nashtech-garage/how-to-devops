# Introduction
![](../../../assets/Screenshot%202024-10-14%20182731.png)


# Installation 
Add the plugin to your frontend app:
```bash
cd packages/app && yarn add @backstage-community/plugin-grafana
```
Configure the plugin in app-config.yaml. The proxy endpoint described below will allow the frontend to authenticate with Grafana without exposing your API key to users. Create an API key if you don't already have one. Viewer access will be enough.
```yaml
# app-config.yaml
proxy:
  endpoints:    
    '/grafana/api':
    # May be a public or an internal DNS
      target: ${GRAFANA_URL}
      headers:
        Authorization: Bearer ${GRAFANA_TOKEN}

grafana:
  # Publicly accessible domain
  domain: ${GRAFANA_URL}
```
Config GRAFANA_URL and GRAFANA_TOKEN in env
```env
export GRAFANA_TOKEN=<your grafana token>
export GRAFANA_URL=<your grafana host>
```
# Display dashboards on a component page
Adding the EntityGrafanaDashboardsCard component to an entity's page will display a list of dashboards related to that entity.

```tsx
// packages/app/src/components/catalog/EntityPage.tsx

import { EntityGrafanaDashboardsCard } from '@backstage-community/plugin-grafana';

// ...

const overviewContent = (
    <Grid item md={6}>
      {/* Grafana dashboard card start */}
      <EntityGrafanaDashboardsCard />
      {/* Grafana alert card end */}
    </Grid>
);
```
Grafana dashboards are correlated to Backstage entities using a selector defined by an annotation added in the entity's catalog-info.yaml file. The EntityGrafanaDashboardsCard component will then display dashboards matching the given selector.

The following selector will return dashboards that have a my-service or a my-service-slo tag and have a generated tag.

```yaml
annotations:
  grafana/dashboard-selector: "(tags @> 'my-service' || tags @> 'my-service-slo') && tags @> 'generated'"
```
Supported variables:

- title: title of the dashboard
- tags: array of tags defined by the dashboard
- url: URL of the dashboard
- folderTitle: title of the folder in which the dashboard is defined
- folderUrl: URL of the folder in which the dashboard is defined

Supported binary operators:

- ||: logical or
- &&: logical and
- ==: equality (=== operator in Javascript)
- !=: inequality (!== operator in Javascript)
- @>: inclusion (left.includes(right) in Javascript)

Supported unary operators:

- !: logical negation

Note that the tags @> "my-service" selector can be simplified as:

# Display alerts on a component page
Adding the EntityGrafanaAlertsCard component to an entity's page will display a list of alerts related to that entity.

```tsx
// packages/app/src/components/catalog/EntityPage.tsx

import { EntityGrafanaAlertsCard } from '@backstage-community/plugin-grafana';

// ...

const overviewContent = (
  <Grid container spacing={3} alignItems="stretch">
    <Grid item md={6}>
      {/* Grafana alert card start */}
      <EntityGrafanaAlertsCard />
      {/* Grafana alert card end */}
    </Grid>
  </Grid>
); 
```
Grafana alerts are correlated to Backstage entities using an annotation added in the entity's catalog-info.yaml file.
## With Grafana Unified Alerting enabled
Open the ArgoCD argocd-cm ConfigMap using the following command:
```yaml
 annotations:
  # This will only use one label selector
  grafana/alert-label-selector: 'label=label-value'
  # OR
  # This will use two label selectors
  grafana/alert-label-selector: 'label1=label-value1,label2=label-value2'
```
## With Grafana Legacy Alerting enabled
If Grafana's Unified Alerting is NOT enabled, alerts are selected by a tag present on the dashboards defining them:
```yaml
annotations:
  grafana/tag-selector: 'my-tag'
```
# Restart ArgoCD Server
To apply the changes, restart the ArgoCD server. Run the following command:
```bash
kubectl rollout restart deployment argocd-server -n argocd
```
# Create grafana component
```yaml
# grafana-component.yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  annotations:
    grafana/alert-label-selector: "rule=cpu,rule=disk-space"
    grafana/dashboard-selector: "(tags @> 'grafanacloud' || tags @> 'cardinality-management')"
  name: grafana
  description: Associate alerts and dashboards to components
spec:
  type: monitoring
  owner: user:default/le.caothihoang
  lifecycle: development
  system: nashtech-idp
```
Add grafana-component.yaml into app-config.yaml
```yaml
    - type: file
      target: ../../packages/backend/component/grafana-component.yaml
      rules:
        - allow: [Component]
```