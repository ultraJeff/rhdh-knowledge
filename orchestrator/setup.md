# RHDH Orchestrator Setup

The Orchestrator enables approval workflows, multi-step provisioning, and automated governance in RHDH. It's powered by **OpenShift Serverless Logic** (the CRDs still use the `sonataflow.org` API group).

## Prerequisites

1. **Install the OpenShift Serverless Logic operator** from OperatorHub
   - Subscription name: `logic-operator-rhel8`
   - Channel: `alpha`
   - Namespace: `openshift-serverless-logic`

2. **Deploy the SonataFlowPlatform** in an infrastructure namespace (e.g., `sonataflow-infra`)

```yaml
apiVersion: sonataflow.org/v1alpha08
kind: SonataFlowPlatform
metadata:
  name: sonataflow-platform
  namespace: sonataflow-infra
spec:
  services:
    dataIndex:
      enabled: true
    jobService:
      enabled: true
```

> **Naming note:** The product is "OpenShift Serverless Logic" but CRDs still use `sonataflow.org/v1alpha08` with kinds `SonataFlow` and `SonataFlowPlatform`.

## RHDH Plugins (4 Required)

```yaml
plugins:
  # Frontend UI
  - package: 'oci://ghcr.io/redhat-developer/rhdh-plugin-export-overlays/red-hat-developer-hub-backstage-plugin-orchestrator:<tag>'
    disabled: false
    pluginConfig:
      dynamicPlugins:
        frontend:
          red-hat-developer-hub.backstage-plugin-orchestrator:
            dynamicRoutes:
              - path: /orchestrator
                importName: OrchestratorPage
                menuItem:
                  icon: scaffolder
                  text: Orchestrator
            mountPoints:
              - mountPoint: entity.page.workflows/cards
                importName: OrchestratorPage
                config:
                  layout:
                    gridColumn: 1 / -1
            entityTabs:
              - path: /workflows
                title: Workflows

  # Backend
  - package: 'oci://ghcr.io/redhat-developer/rhdh-plugin-export-overlays/red-hat-developer-hub-backstage-plugin-orchestrator-backend:<tag>'
    disabled: false

  # Form widgets
  - package: 'oci://ghcr.io/redhat-developer/rhdh-plugin-export-overlays/red-hat-developer-hub-backstage-plugin-orchestrator-form-widgets:<tag>'
    disabled: false

  # Scaffolder bridge
  - package: 'oci://ghcr.io/redhat-developer/rhdh-plugin-export-overlays/red-hat-developer-hub-backstage-plugin-scaffolder-backend-module-orchestrator:<tag>'
    disabled: false
```

## App-Config

```yaml
orchestrator:
  dataIndexService:
    url: http://sonataflow-platform-data-index-service.sonataflow-infra
  sonataflow:
    baseUrl: http://sonataflow-platform-data-index-service.sonataflow-infra
```

## RBAC

```csv
p, role:default/plugins, orchestrator.workflow, read, allow
p, role:default/plugins, orchestrator.workflow.use, update, allow
```

## OCI Plugin Tags

Tags follow the pattern `bs_<backstage-version>__<plugin-version>`. Query the registry:

```bash
skopeo list-tags docker://ghcr.io/redhat-developer/rhdh-plugin-export-overlays/<plugin-name>
```

Never guess tags from npm versions — they won't match.

## Docs

- [RHDH 1.10 — Orchestrator](https://docs.redhat.com/en/documentation/red_hat_developer_hub/1.10/html-single/orchestrator_in_red_hat_developer_hub/index)
- [Orchestrator community](https://rhdhorchestrator.io)
