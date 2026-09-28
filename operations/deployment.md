# RHDH Deployment Notes

## Operator Deployment

RHDH is deployed via the RHDH operator. Subscription channel: `fast-1.8` (or current).

## Key Config Files

Managed by ArgoCD as the `developer-hub` application:

- `app-config-production.yaml` — main Backstage config (ConfigMap)
- `dynamic-plugins.yaml` — OCI-based dynamic plugin loading (ConfigMap)
- `rbac-policies.yaml` — RHDH RBAC policies (ConfigMap)

ConfigMaps use these annotations/labels:
```yaml
annotations:
  rhdh.redhat.com/backstage-name: developer-hub
  argocd.argoproj.io/sync-wave: "2"
labels:
  rhdh.redhat.com/ext-config-sync: "true"
```

## Secrets Management

Recommended: External Secrets Operator (ESO) syncing from Vault or another secrets manager.

Common secrets needed:
- Keycloak credentials (clientId, clientSecret)
- ArgoCD credentials (username, password, URL)
- GitHub/GitLab PAT and app credentials
- Backend secret (session secret)
- AAP token
- Registry auth

## RHDH-isms

- Environment secrets use `${{ environment.secrets.* }}` (NOT upstream Backstage's `${{ secrets.* }}`)
- Never guess dynamic plugin OCI tags from npm versions — query the GHCR API
- `policyFileReload: true` enables hot-reload of RBAC CSV

## Post-Upgrade Checklist

After OCP minor version upgrades:
1. Check operator subscription channels for `ResolutionFailed`
2. Delete stale CSVs from old versions
3. Verify StorageClass default annotation
4. Check image pruner CronJob for failures

## Docs

- [RHDH 1.10 — Dynamic plugin configuration](https://docs.redhat.com/en/documentation/red_hat_developer_hub/1.10/html-single/configuring_plugins_in_red_hat_developer_hub/index)
