# Introduction
This plugin uses the Backstage proxy to securely communicate with New Relic's APIs. We use NerdGraph (New Relic's GraphQL API)

![](../../../assets/Screenshot%202024-10-15%20143825.png)

# Installation 
Add the plugin to your frontend app:
```bash
cd packages/app && yarn add @backstage-community/plugin-newrelic-dashboard
```
Add the following to your app-config.yaml to enable this configuration
```yaml
# app-config.yaml
proxy:
  endpoints:    
    '/newrelic/api':
      target: https://api.newrelic.com
      headers:
        X-Api-Key: ${NEW_RELIC_USER_KEY}
```
Config NEW_RELIC_USER_KEY in env
```env
export NEW_RELIC_USER_KEY=<your new relic uer key>

```
To generate a New Relic API Key , you can visit this [link](https://one.newrelic.com/api-keys?state=bf6c50f8-8572-ce04-ca42-d7fb485f4985)

# Add the following to EntityPage.tsx to display New Relic Dashboard Tab
```tsx
/// In packages/app/src/components/catalog/EntityPage.tsx

import {
  isNewRelicDashboardAvailable,
  EntityNewRelicDashboardContent,
  EntityNewRelicDashboardCard,
} from '@backstage-community/plugin-newrelic-dashboard';

// ...
const monitoringPage = (
  <EntityLayout>
    <EntityLayout.Route path="/" title="Overview">
    <Grid container spacing={3} alignItems="stretch">
    {entityWarningContent}
    <Grid item md={6}>
      <EntityAboutCard variant="gridItem" />
    </Grid>  
  </Grid>
    </EntityLayout.Route>
   <EntityLayout.Route
      if={isNewRelicDashboardAvailable}
      path="/newrelic-dashboard"
      title="New Relic Dashboard"
    >
      <EntityNewRelicDashboardContent />
    </EntityLayout.Route>
  
  </EntityLayout>
);

```

Add newrelic.com/dashboard-guid annotation in catalog descriptor file

```yaml
annotations:
  newrelic.com/dashboard-guid: dashboard_guid>
```
To Obtain the dashboard's GUID: Click the info icon by the dashboard's name to access the See metadata and manage tags modal and see the dashboard's GUID.
# Update monitoring-component.yaml if's existing
```yaml
# monitoring-component.yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  annotations:
    newrelic.com/dashboard-guid: NDc5NDk1NnxWSVp8REFTSEJPQVJEfGRhOjcxMzEwNzA
    grafana/alert-label-selector: "rule=cpu,rule=disk-space"
    grafana/dashboard-selector: "(tags @> 'grafanacloud' || tags @> 'cardinality-management')"
  name: monitoring
  description: Easily view your New Relic Dashboards, Grafana Dashboards in Backstage, via real-time snapshots of your dashboards
spec:
  type: monitoring
  owner: user:default/le.caothihoang
  lifecycle: development
  system: nashtech-idp
```
