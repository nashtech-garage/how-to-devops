# Setup Organization 
## Define user
```yaml
# user.yaml
# https://backstage.io/docs/features/software-catalog/descriptor-format#kind-user
apiVersion: backstage.io/v1alpha1
kind: User
metadata:
  name: le.caothihoang
  annotations:
    github.com/user-login: le.caothihoang
spec:
  profile:
    displayName: Le Cao
    email: le.caothihoang@gmail.com
    picture: https://avatars.githubusercontent.com/hoanglecao
  memberOf: [platform-engineering]

---
apiVersion: backstage.io/v1alpha1
kind: User
metadata:
  name: lecao_nashtechglobal.com
  annotations:
    microsoft.com/email: lecao@nashtechglobal.com
    backstage.io/normalized-identity: lecao_nashtechglobal.com#ext#
spec:
  profile:
    displayName: Le Cao Thi Hoang
    email: lecao@nashtechglobal.com
  memberOf: [cloud-architects]
```
## Define group
```yaml
# https://backstage.io/docs/features/software-catalog/descriptor-format#kind-group
# group.yaml
apiVersion: backstage.io/v1alpha1
kind: Group
metadata:
  namespace: default
  name: platform-engineering
  description: The platform engineering team - A platform engineering team often includes roles such as platform engineers, site reliability engineers, cloud architects, and security engineers. Other roles might include DevOps engineers, automation engineers, and quality assurance engineers
spec:
  type: team
  profile:
    displayName: Platform Engineering Team 
    email: platform-engineering@nashtechglobal.com
  children: []
  members: [user:default/le.caothihoang]

---
apiVersion: backstage.io/v1alpha1
kind: Group
metadata:
  namespace: default
  name: platform-engineers
  description: The platform engineers Team - Platform engineers are responsible for designing, building, and maintaining the platform infrastructure.
spec:
  type: team
  profile:
    displayName: Platform engineers Team
    email: platform-engineers@nashtechglobal.com
  children: []
  parent: platform-engineering

---
apiVersion: backstage.io/v1alpha1
kind: Group
metadata:
  namespace: default
  name: site-reliability-engineers
  description: Site reliability engineers (SREs) Team - Site reliability engineers (SREs) handle deployments to superior environments, are experienced in incident management and response, and have strong problem-solving skills.
spec:
  type: team
  profile:
    displayName: Site reliability engineers (SREs) Team
    email: site-reliability-engineers@nashtechglobal.com
  children: []
  parent: platform-engineering

---
apiVersion: backstage.io/v1alpha1
kind: Group
metadata:
  namespace: default
  name: cloud-architects
  description: Cloud architects Team -  should be experts in designing platforms and ensuring the platform architecture is resilient and satisfies all the organization’s needs.
spec:
  type: team
  profile:
    displayName: Cloud architects Team
    email: cloud-architects@nashtechglobal.com
  children: []
  parent: platform-engineering

---
apiVersion: backstage.io/v1alpha1
kind: Group
metadata:
  namespace: default
  name: security-engineers
  description: Security engineers Team -  Security engineers work hand in hand with platform engineers and cloud architects to implement and maintain the security best practices for the platform.
spec:
  type: team
  profile:
    displayName: Security engineers Team
    email: security-engineers@nashtechglobal.com
  children: []
  parent: platform-engineering

```

## Setup Org

```yaml
# org.yaml
apiVersion: backstage.io/v1alpha1
kind: Location
metadata:
  name: my-org
  description: Organization structure for NashTech
spec:
  targets:
    - ./user.yaml
    - ./group.yaml

```
# Setup RBAC
Using @janus-idp/backstage-plugin-rbac-backend package
https://www.npmjs.com/package/@janus-idp/backstage-plugin-rbac

## Create rbac-policy
```csv
# rbac-policy.csv
# Assigning users to roles
g, user:default/le.caothihoang, role:default/catalog-admin
g, user:default/le.caothihoang, role:default/admin-rbac
g, user:default/le.caothihoang, role:default/scaffolder-admin
g, user:default/le.caothihoang, role:default/plugin-admin
g, user:default/lecao_nashtechglobal.com, role:default/developer

# Assigning groups to roles
g, group:default/platform-engineering, role:default/admin-catalog
g, group:default/cloud-architects, role:default/developer

# Defining permissions for developer role
p, role:default/developer, catalog.entity.read, read, allow
p, role:default/developer, catalog.entity.create, create, allow

# Catalog entity permissions
p, role:default/catalog-admin, catalog.entity.read, read, allow
p, role:default/catalog-admin, catalog.entity.create, create, allow
p, role:default/catalog-admin, catalog.entity.refresh, update, allow
p, role:default/catalog-admin, catalog.entity.delete, delete, allow

# Catalog location permissions
p, role:default/catalog-admin, catalog.location.read, read, allow
p, role:default/catalog-admin, catalog.location.create, create, allow
p, role:default/catalog-admin, catalog.location.delete, delete, allow

# RBAC (Role-Based Access Control)
p, role:default/admin-rbac, policy.entity.create, create, allow
p, role:default/admin-rbac, policy.entity.read, read, allow
p, role:default/admin-rbac, policy.entity.update, update, allow
p, role:default/admin-rbac, policy.entity.delete, delete, allow

# Scaffolder permissions
p, role:default/scaffolder-admin, scaffolder.template, read, allow
p, role:default/scaffolder-admin, scaffolder.action.execute, use, allow
p, role:default/scaffolder-admin, scaffolder.task.create, create, allow
p, role:default/scaffolder-admin, scaffolder.task.read, read, allow
p, role:default/scaffolder-admin, scaffolder.task.cancel, use, allow
p, role:default/scaffolder-admin, scaffolder.template.parameter.read, read, allow
p, role:default/scaffolder-admin, scaffolder.template.step.read, read, allow

# Plugins
p, role:default/plugin-admin, argocd.view.read, read, allow
```

## Create condition-policies (optional)

```yaml
# condition-policies.yaml
result: CONDITIONAL
roleEntityRef: 'role:default/admin-catalog'
pluginId: catalog
resourceType: catalog-entity
permissionMapping:
  - delete
  - update
conditions:
  rule: IS_ENTITY_OWNER
  resourceType: catalog-entity
  params:
    claims:
      - 'user:default/le.caothihoang'

```
##  Setup permission in app-config.yaml

```yaml
permission:
  enabled: true  
  rbac:
    policies-csv-file: ../../packages/backend/org/rbac-policy.csv
    #conditionalPoliciesFile: ../../packages/backend/org/conditional-policies.yaml
    policyFileReload: true
```

