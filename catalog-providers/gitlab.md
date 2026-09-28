# GitLab Catalog Provider

Auto-discovers `catalog-info.yaml` files across a GitLab instance and registers them in the RHDH catalog.

## What is catalog-info.yaml?

A metadata file in the root of each repo that describes the component to RHDH. Without this file, the discovery provider has nothing to find.

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: my-service
  description: A backend service
  annotations:
    gitlab.com/project-slug: my-org/my-service
    argocd/app-name: my-service
spec:
  type: service
  lifecycle: production
  owner: group:default/platform-team
```

Best practice: include `catalog-info.yaml` in software templates so new projects are auto-registered.

## Integration Config

```yaml
integrations:
  gitlab:
    - host: gitlab.example.com
      token: ${GITLAB_PAT}
      apiBaseUrl: https://gitlab.example.com/api/v4
```

PAT needs `read_api` at minimum. For templates creating repos: `api` scope.

## Discovery Plugin

Plugin (preinstalled, disabled by default):

```yaml
plugins:
  - package: ./dynamic-plugins/dist/backstage-plugin-catalog-backend-module-gitlab-dynamic
    disabled: false
```

```yaml
catalog:
  providers:
    gitlab:
      default:
        host: gitlab.example.com
        branch: main
        fallbackBranch: master
        skipForkedRepos: false
        entityFilename: catalog-info.yaml
        # group: my-org/platform-team
        # projectPattern: '[\s\S]*'
        schedule:
          frequency:
            minutes: 30
          timeout:
            minutes: 10
          initialDelay:
            seconds: 15
```

## GitLab Org Provider (User/Group Sync)

Separate plugin for importing GitLab users and groups:

```yaml
plugins:
  - package: ./dynamic-plugins/dist/backstage-plugin-catalog-backend-module-gitlab-org-dynamic
    disabled: false
```

```yaml
catalog:
  providers:
    gitlab:
      default:
        host: gitlab.example.com
        orgEnabled: true
        group: my-org
        restrictUsersToGroup: false
        schedule:
          frequency:
            minutes: 30
          timeout:
            minutes: 3
          initialDelay:
            seconds: 15
```

## Scaffolder Actions

For software templates that publish repos to GitLab:

```yaml
plugins:
  - package: ./dynamic-plugins/dist/backstage-plugin-scaffolder-backend-module-gitlab-dynamic
    disabled: false
```

Provides: `publish:gitlab`, `publish:gitlab:merge-request`, `gitlab:group:ensureExists`.

## Gotchas

- Use `entityFilename` (not `catalogPath` — that's GitHub-only)
- `group` filter is optional for self-managed GitLab but **required for gitlab.com**
- `fallbackBranch` is more reliable than `branch` — it kicks in when a repo doesn't have the configured default branch
- `orgEnabled: true` without `group` on gitlab.com will throw an error
- Self-managed GitLab with custom TLS certs may need the CA added to the RHDH pod trust store

## Docs

- [RHDH 1.10 — GitLab integration](https://docs.redhat.com/en/documentation/red_hat_developer_hub/1.10/html-single/configuring_plugins_in_red_hat_developer_hub/index)
- [Backstage GitLab Discovery](https://backstage.io/docs/integrations/gitlab/discovery/)
- [Backstage GitLab Org](https://backstage.io/docs/integrations/gitlab/org/)
