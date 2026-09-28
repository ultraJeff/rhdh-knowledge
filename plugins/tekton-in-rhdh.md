# Tekton CI in RHDH

Scope: How RHDH surfaces Tekton pipeline data on entity pages. Not a Tekton setup guide.

## Plugin

```yaml
plugins:
  - package: 'oci://ghcr.io/redhat-developer/rhdh-plugin-export-overlays/backstage-community-plugin-tekton:<tag>'
    disabled: false
```

## RBAC

```csv
p, role:default/plugins, tekton.view.read, read, allow
```

## How It Works

The Tekton plugin reads PipelineRun and TaskRun resources from the Kubernetes API (via the Kubernetes backend plugin). It shows pipeline status, history, and logs on entity pages.

## Prerequisites

- Kubernetes backend plugin enabled and configured
- `customResources` configured for Tekton CRDs:

```yaml
kubernetes:
  customResources:
    - group: tekton.dev
      apiVersion: v1
      plural: pipelineruns
    - group: tekton.dev
      apiVersion: v1
      plural: taskruns
```

## Entity Annotation

Components need the appropriate annotation to link to their Tekton pipelines. The Tekton plugin discovers PipelineRuns by matching labels on the cluster.
