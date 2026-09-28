# RHDH Catalog Modeling

## Entity Kinds

| Kind | Purpose | Example |
|------|---------|---------|
| Component | A piece of software (service, website, library) | `my-api-service` |
| API | An interface exposed by a Component | REST, gRPC, GraphQL |
| Resource | Infrastructure a Component depends on | Database, S3 bucket, queue |
| System | A collection of Components and Resources | `payment-system` |
| Domain | A business domain grouping Systems | `finance`, `logistics` |
| Group | An organizational team | `platform-engineering` |
| User | An individual person | `jane.doe` |
| Template | A software template for scaffolding | `astro-app` |
| Location | A pointer to other entity definitions | `all.yaml` |

## Org Data

Define teams as Group entities in a Git repo:

```yaml
apiVersion: backstage.io/v1alpha1
kind: Group
metadata:
  name: platform-engineering
  description: Platform Engineering team
spec:
  type: team
  children: []
```

Register via a Location entity alongside templates.

For production, sync groups from your IdP via a catalog provider (Keycloak, Okta, GitLab org) rather than maintaining static YAML.

## Entity Annotations

Annotations connect catalog entities to external systems. Key annotations:

```yaml
metadata:
  annotations:
    # Source control
    github.com/project-slug: org/repo-name
    gitlab.com/project-slug: org/repo-name
    backstage.io/source-location: url:https://github.com/org/repo

    # CI/CD
    argocd/app-name: my-app
    tektonci/build-namespace: my-app-ci

    # Monitoring
    backstage.io/techdocs-ref: dir:.

    # Image registries
    quay.io/repository-slug: org/image-name
    nexus-repository-manager/docker.image-name: org/image-name
```

These annotations are what enable plugins to show relevant data on entity pages — ArgoCD shows deployments, Tekton shows pipeline runs, etc.

## Ownership Model

Every entity should have an `owner` in its spec pointing to a Group or User entity:

```yaml
spec:
  owner: group:default/platform-engineering
  # or
  owner: user:default/admin
```

Ownership drives:
- Visibility in the catalog (filtering by "owned by me")
- RBAC policy resolution (group-based permissions)
- Notifications routing in orchestrator workflows

## Component Types

The `spec.type` field is freeform but conventional values include:

- `service` — backend service, API
- `website` — frontend application
- `library` — shared library
- `infrastructure` — cloud resource, database
- `documentation` — docs-only component

Templates set this via `spec.type` on the Template entity itself, which also controls how the template appears in the "Create" page.

## catalog-info.yaml in Scaffolded Projects

Always include a templated `catalog-info.yaml` in your software template skeletons so new projects are automatically discoverable by the catalog provider:

```yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: ${{ values.name }}
  description: ${{ values.description }}
  annotations:
    github.com/project-slug: ${{ values.ghOwner }}/${{ values.repoName }}
    argocd/app-name: ${{ values.name }}
spec:
  type: service
  lifecycle: production
  owner: ${{ values.owner }}
```

## Lifecycle Values

- `experimental` — early stage, not production-ready
- `production` — actively maintained and serving traffic
- `deprecated` — being phased out
