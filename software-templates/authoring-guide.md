# Software Template Authoring Guide

## Template Structure

A template is a `scaffolder.backstage.io/v1beta3` `Template` entity with three main sections: `parameters` (user inputs), `steps` (actions to execute), and `output` (links shown after completion).

Templates live in a Git repo and are registered via a `Location` entity (e.g., `all.yaml`) that points to each `template.yaml`.

## Template Registration

Use a single `all.yaml` Location entity to register all templates and org data:

```yaml
apiVersion: backstage.io/v1alpha1
kind: Location
metadata:
  name: rhdh-software-templates
  description: Custom software templates for RHDH
spec:
  type: url
  targets:
    - ./org/platform-engineering.yaml
    - ./astro-app/template.yaml
    - ./postgres-db/template.yaml
    # ...
```

Register this Location in RHDH's app-config:

```yaml
catalog:
  locations:
    - type: url
      target: https://raw.githubusercontent.com/<org>/rhdh-software-templates/main/all.yaml
      rules:
        - allow: [Template, Group]
```

## Template Categories

Templates fall into two main types based on `spec.type`:

### Application Templates (`type: service`)

Scaffold full application projects. Typical flow:
1. `fetch:template` — source skeleton + gitops skeleton
2. `publish:github` (or `publish:gitlab`) — create both repos
3. `argocd:create-resources` — set up ArgoCD app + project
4. `catalog:register` — register component in RHDH catalog

Example: astro-app, grpc-quarkus, camel-adapter

### Infrastructure Templates (`type: infrastructure`)

Provision cloud or platform resources. May use:
- `http:backstage:request` to call APIs
- AAP scaffolder actions to trigger Ansible job templates
- Crossplane/Terraform provisioning (TODO pattern for stubs)
- `debug:log` as placeholder for unimplemented steps

Example: ec2-instance, s3-bucket, postgres-db, service-mesh-join

## Parameters

### Common UI Fields

```yaml
# Entity name with validation
name:
  type: string
  ui:field: EntityNamePicker
  maxLength: 63
  pattern: '^[a-z]([-a-z0-9]*[a-z0-9])?$'

# Owner picker (groups and users from catalog)
owner:
  type: string
  ui:field: OwnerPicker
  ui:options:
    catalogFilter:
      kind: [Group, User]

# Enum dropdown
instance_type:
  type: string
  enum: [t3.micro, t3.small, t3.medium]

# Checkbox array
cloud_providers:
  type: array
  uniqueItems: true
  minItems: 1
  items:
    type: string
    enum: [aws, azure, gcp]
  ui:widget: checkboxes

# Boolean toggle
assign_public_ip:
  type: boolean
  default: false

# Integer with bounds
storage_size_gb:
  type: integer
  default: 20
  minimum: 8
  maximum: 1000
```

### Multi-Page Parameters

Split parameters into multiple pages using an array:

```yaml
parameters:
  - title: Application Information
    required: [name, owner]
    properties:
      name: ...
      owner: ...
  - title: Deployment Information
    required: [imageRegistry]
    properties:
      imageRegistry: ...
```

### Conditional Values

Use inline ternary expressions for optional fields:

```yaml
namespace: ${{ parameters.namespace if parameters.namespace else parameters.name }}
repoName: ${{ parameters.repoName if parameters.repoName else parameters.name + '-gitops' }}
```

## Steps

### Core Actions (Preinstalled)

| Action | Purpose |
|--------|---------|
| `fetch:template` | Copy and template a skeleton directory |
| `publish:github` | Create a GitHub repo |
| `publish:gitlab` | Create a GitLab repo |
| `catalog:register` | Register a component in the RHDH catalog |
| `debug:log` | Log a message (useful as placeholder) |
| `debug:wait` | Pause execution (use between publish and ArgoCD) |

### Plugin Actions (Require Dynamic Plugins)

| Action | Plugin Required |
|--------|----------------|
| `argocd:create-resources` | `roadiehq-scaffolder-backend-argocd` |
| `http:backstage:request` | `roadiehq-scaffolder-backend-module-http-request` |
| `roadiehq:utils:*` | `roadiehq-scaffolder-backend-module-utils` |
| `publish:gitlab:merge-request` | `backstage-plugin-scaffolder-backend-module-gitlab-dynamic` |

### The GitOps Publish Pattern

Most app templates create two repos — source and gitops:

```yaml
steps:
  - id: fetch-source
    action: fetch:template
    input:
      url: ./content
      targetPath: source
      values: { ... }

  - id: fetch-gitops
    action: fetch:template
    input:
      url: ./gitops
      targetPath: gitops
      values: { ... }

  - id: publish-source
    action: publish:github
    input:
      sourcePath: source
      repoUrl: ${{ parameters.ghHost }}?owner=${{ parameters.ghOwner }}&repo=${{ parameters.name }}

  - id: publish-gitops
    action: publish:github
    input:
      sourcePath: gitops
      repoUrl: ${{ parameters.ghHost }}?owner=${{ parameters.ghOwner }}&repo=${{ parameters.name + '-gitops' }}

  - id: wait-for-repos
    action: debug:wait
    input:
      seconds: 3

  - id: create-argocd-resources
    action: argocd:create-resources
    input:
      appName: ${{ parameters.name }}
      argoInstance: argocd
      projectName: ${{ parameters.name }}  # MUST be unique — 'default' fails
      namespace: ${{ parameters.namespace if parameters.namespace else parameters.name }}
      repoUrl: https://${{ parameters.ghHost }}/${{ parameters.ghOwner }}/${{ parameters.name + '-gitops' }}.git
      path: ./k8s/overlays/development
```

### Important: ArgoCD Resources

- Each ArgoCD app needs a **unique AppProject name** — using `default` fails
- `CreateNamespace=false` is hardcoded in the Roadie plugin, so gitops repos must include Namespace manifests
- Add a `debug:wait` (2-3 seconds) between `publish:github` and `argocd:create-resources` for repo availability

## Output

```yaml
output:
  links:
    - title: Source Repository
      url: ${{ steps['publish-source'].output.remoteUrl }}
      icon: github
    - title: Open Component in Catalog
      icon: catalog
      entityRef: ${{ steps['register-source'].output.entityRef }}
```

## Skeleton Directory Layout

```
my-template/
  template.yaml          # Template definition
  content/               # Source code skeleton (fetch:template url: ./content)
    catalog-info.yaml    # Backstage entity (templated)
    src/
    Dockerfile
  gitops/                # GitOps manifests skeleton (fetch:template url: ./gitops)
    catalog-info.yaml    # Backstage entity for gitops resource
    k8s/
      overlays/
        development/
```

## AAP Integration in Templates

Use environment secrets to pass the AAP token:

```yaml
scaffolder:
  defaultEnvironment:
    secrets:
      AAP_TOKEN: ${AAP_TOKEN}
```

Templates reference it as `${{ environment.secrets.AAP_TOKEN }}` (RHDH syntax, NOT upstream `${{ secrets.* }}`).

## Template for catalog-info.yaml

Always include a `catalog-info.yaml` in your skeleton so scaffolded projects are auto-discoverable:

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
