# Introduction
Prometheus plugin provides visualization of Prometheus metrics and alerts

![](../../../assets/Screenshot%202024-10-16%20132225.png)

# Installation 
The Prometheus plugin is a frontend plugin. You will need to install it, configure it and add it to an appropriate location on the entity page.

Install the Prometheus plugin into Backstage from the app folder of your repository.
```bash
cd packages/app
yarn add @roadiehq/backstage-plugin-prometheus
```
Add the following to your app-config.yaml to enable this configuration
```yaml
# app-config.yaml
proxy:
  endpoints:    
    '/prometheus/api':
      # url to the api and path of your hosted prometheus instance
      target: ${PROMETHEUS_URL}
      changeOrigin: true
      headers:
        Authorization: Basic ${PROMETHEUS_TOKEN}
```
Config PROMETHEUS_URL, PROMETHEUS_TOKEN in env
```env
export PROMETHEUS_TOKEN=<your token>
export PROMETHEUS_URL=<your url>

```
To generate a New Relic API Key , you can visit this [link](https://roadie.io/backstage/plugins/prometheus/)

# Content page , Widget setup
```tsx
// packages/app/src/components/catalog/EntityPage.tsx

import {
  EntityPrometheusContent,
  EntityPrometheusGraphCard,
  EntityPrometheusAlertCard
} from '@roadiehq/backstage-plugin-prometheus';

// ...
const monitoringPage = (
  <EntityLayout>
          <EntityLayout.Route path="/prometheus" title="Prometheus">
        <EntityPrometheusContent />
      </EntityLayout.Route>
        <EntityLayout.Route path="/prometheus-alert" title="Prometheus Alert">
        <Grid item md={8}>
        <EntityPrometheusAlertCard />
        </Grid>
      </EntityLayout.Route>
      <EntityLayout.Route path="/prometheus-graph" title="Prometheus Graph">
        <Grid item md={6}>
        <EntityPrometheusGraphCard />
  </EntityLayout>
);

```

Add newrelic.com/dashboard-guid annotation in catalog descriptor file

```yaml
annotations:
  prometheus.io/rule: <rule>
  prometheus.io/alert: <alert>
```

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
    prometheus.io/rule: memUsage|component
    prometheus.io/alert: all
  name: monitoring
  description: Easily view your New Relic Dashboards, Grafana Dashboards in Backstage, via real-time snapshots of your dashboards
spec:
  type: monitoring
  owner: user:default/le.caothihoang
  lifecycle: development
  system: nashtech-idp
```
