# RHDH RBAC Configuration

## Enable RBAC

```yaml
permission:
  enabled: true
  rbac:
    policies-csv-file: /opt/app-root/src/rbac-policies.csv
    policyFileReload: true
    admin:
      users:
        - name: user:default/<admin-user>
    pluginsWithPermission:
      - catalog
      - scaffolder
      - permission
      - orchestrator
```

## RBAC UI Plugin

Preinstalled, disabled by default:

```yaml
plugins:
  - package: ./dynamic-plugins/dist/backstage-community-plugin-rbac
    disabled: false
```

## Policy CSV Format

Two line types:
- `p` lines: assign permissions to roles
- `g` lines: assign users/groups to roles

## Example Policies

```csv
# ----- Role: read-only catalog access -----
p, role:default/viewer, catalog.entity.read, read, allow
p, role:default/viewer, catalog.location.read, read, allow

# ----- Role: template execution -----
p, role:default/developer, catalog.entity.read, read, allow
p, role:default/developer, catalog.location.read, read, allow
p, role:default/developer, scaffolder.action.execute, use, allow
p, role:default/developer, scaffolder.template.parameter.read, read, allow
p, role:default/developer, scaffolder.template.step.read, read, allow
p, role:default/developer, scaffolder.task.create, create, allow
p, role:default/developer, scaffolder.task.cancel, use, allow
p, role:default/developer, scaffolder.task.read, read, allow
p, role:default/developer, catalog.location.create, create, allow

# ----- Role: platform engineer (full catalog management) -----
p, role:default/platformengineer, catalog.entity.create, create, allow
p, role:default/platformengineer, catalog.entity.refresh, update, allow
p, role:default/platformengineer, catalog.entity.delete, delete, allow
p, role:default/platformengineer, catalog.location.create, create, allow
p, role:default/platformengineer, catalog.location.delete, delete, allow

# ----- Role: plugin visibility -----
p, role:default/plugins, topology.view.read, read, allow
p, role:default/plugins, tekton.view.read, read, allow
p, role:default/plugins, argocd.view.read, read, allow
p, role:default/plugins, kubernetes.proxy, use, allow
p, role:default/plugins, orchestrator.workflow, read, allow
p, role:default/plugins, orchestrator.workflow.use, update, allow

# ----- Role: admin plugins -----
p, role:default/admin_plugins, kubernetes.clusters.read, read, allow
p, role:default/admin_plugins, kubernetes.resources.read, read, allow
p, role:default/admin_plugins, extensions.plugin.configuration.read, read, allow
p, role:default/admin_plugins, extensions.plugin.configuration.write, create, allow
p, role:default/admin_plugins, extensions.plugin.configuration.delete, delete, allow

# ----- Assign groups to roles -----
g, group:default/rhdh-developers, role:default/viewer
g, group:default/rhdh-developers, role:default/developer
g, group:default/rhdh-developers, role:default/plugins
g, group:default/rhdh-platform-team, role:default/viewer
g, group:default/rhdh-platform-team, role:default/developer
g, group:default/rhdh-platform-team, role:default/platformengineer
g, group:default/rhdh-platform-team, role:default/plugins
```

## Key Gotchas

### Orchestrator Permission Names (Changed in RHDH 1.9+)

The permission name is `orchestrator.workflow` with action `read` — NOT `orchestrator.workflow.read` as a single permission name. Similarly, `orchestrator.workflow.use` with action `update`.

Per-workflow permissions: `orchestrator.workflow.<workflowId>` with action `read`, and `orchestrator.workflow.use.<workflowId>` with action `update`.

### pluginsWithPermission

Controls which plugins participate in the permission framework. If a plugin isn't listed, its resources are accessible to everyone by default.

### Best Practices

- Assign roles to **groups**, not individual users
- `policyFileReload: true` lets you update the CSV in a ConfigMap without restarting the pod
- Always verify permission names against official docs — they change between versions

## Docs

- [RHDH 1.10 — Authorization and RBAC](https://docs.redhat.com/en/documentation/red_hat_developer_hub/1.10/html-single/authorization_in_red_hat_developer_hub/index)
- [RHDH 1.9 Orchestrator RBAC](https://docs.redhat.com/en/documentation/red_hat_developer_hub/1.9/html-single/orchestrator_in_red_hat_developer_hub/index)
