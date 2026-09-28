# Recommended RHDH Dynamic Plugins

## Software Templates (Scaffolder Actions)

| Plugin | Purpose |
|--------|---------|
| `backstage-plugin-scaffolder-backend-module-github-dynamic` | `publish:github` — create repos from templates (GitHub) |
| `backstage-plugin-scaffolder-backend-module-gitlab-dynamic` | `publish:gitlab`, `gitlab:repo:push` — create repos from templates (GitLab) |
| `roadiehq-scaffolder-backend-module-utils` | Utility actions: `roadiehq:utils:jsonata`, `roadiehq:utils:merge`, `roadiehq:utils:serialize:yaml` |
| `roadiehq-scaffolder-backend-module-http-request` | `http:backstage:request` — HTTP calls from templates |
| `roadiehq-scaffolder-backend-argocd` | `argocd:create-resources` — create ArgoCD apps from templates |
| `backstage-community-backstage-plugin-scaffolder-backend-module-mta` | MTA scaffolder actions |
| `ansible-plugin-scaffolder-backend-module-backstage-rhaap` | AAP scaffolder actions — launch job templates |

## Orchestrator

| Plugin | Purpose |
|--------|---------|
| `red-hat-developer-hub-backstage-plugin-orchestrator` | Frontend UI |
| `red-hat-developer-hub-backstage-plugin-orchestrator-backend` | Backend API |
| `red-hat-developer-hub-backstage-plugin-orchestrator-form-widgets` | Custom form widgets |
| `red-hat-developer-hub-backstage-plugin-scaffolder-backend-module-orchestrator` | Scaffolder-orchestrator bridge |

## CI/CD Visibility

| Plugin | Purpose |
|--------|---------|
| `backstage-community-plugin-argocd` + backend | ArgoCD deployment status on entity pages |
| `backstage-community-plugin-tekton` | Tekton PipelineRun/TaskRun visibility |
| `backstage-community-plugin-topology` | OpenShift topology view |
| `backstage-plugin-kubernetes` + backend | K8s resource viewer |

## Identity & Access

| Plugin | Purpose |
|--------|---------|
| `backstage-community-plugin-catalog-backend-module-keycloak-dynamic` | Keycloak user/group sync |
| `backstage-plugin-catalog-backend-module-github-dynamic` | GitHub catalog discovery |
| `backstage-plugin-catalog-backend-module-github-org-dynamic` | GitHub org user/group sync |
| `backstage-plugin-catalog-backend-module-gitlab-dynamic` | GitLab catalog discovery |
| `backstage-plugin-catalog-backend-module-gitlab-org-dynamic` | GitLab org user/group sync |
| `backstage-community-plugin-rbac` | RBAC management UI |

## Notifications

| Plugin | Purpose |
|--------|---------|
| `backstage-plugin-notifications` + backend | In-app notification system |
| `backstage-plugin-signals` + backend | Real-time signals |

## Image Registries

| Plugin | Purpose |
|--------|---------|
| `backstage-community-plugin-quay` | Quay image registry |
| `backstage-community-plugin-nexus-repository-manager` | Nexus repository manager |

## Ansible

| Plugin | Purpose |
|--------|---------|
| `ansible-plugin-backstage-rhaap` | Ansible page + UI |
| `ansible-plugin-backstage-self-service` | AAP field extensions for templates |
| `ansible-backstage-plugin-catalog-backend-module-rhaap` | AAP catalog backend |

## Plugin Loading

- **Preinstalled plugins** use `./dynamic-plugins/dist/` path — just set `disabled: false`
- **OCI plugins** use `oci://ghcr.io/redhat-developer/rhdh-plugin-export-overlays/` prefix
- **Red Hat registry plugins** (Ansible) use `oci://registry.redhat.io/`
- Find OCI tags: `skopeo list-tags docker://ghcr.io/redhat-developer/rhdh-plugin-export-overlays/<plugin-name>`
- Don't enable plugins you're not ready to configure

## ArgoCD Plugin Notes

The Roadie ArgoCD plugin has been replaced by the community ArgoCD plugins loaded via OCI. Disable the old defaults:

```yaml
- package: ./dynamic-plugins/dist/roadiehq-backstage-plugin-argo-cd-backend-dynamic
  disabled: true
- package: ./dynamic-plugins/dist/backstage-community-plugin-redhat-argocd
  disabled: true
```

The Roadie ArgoCD scaffolder action is still needed for `argocd:create-resources` in templates (community alpha actions API not compatible with scaffolder registry in RHDH 1.10).

## Template GitOps Flow

```
RHDH Template -> fetch:template -> publish:github/gitlab (source + gitops repos)
             -> argocd:create-resources (creates ArgoCD App + AppProject)
             -> ArgoCD syncs gitops repo -> cluster
```

Each ArgoCD app needs a unique AppProject name (`default` fails). `CreateNamespace=false` is hardcoded in the Roadie plugin, so gitops repos should include Namespace manifests.
