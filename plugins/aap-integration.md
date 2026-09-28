# RHDH + Ansible Automation Platform Integration

## Overview

RHDH integrates with AAP through 5 dynamic plugins that provide the Ansible page, scaffolder field extensions, catalog sync, and scaffolder actions.

## Dynamic Plugins

```yaml
plugins:
  # Ansible page + UI
  - package: 'oci://registry.redhat.io/ansible-automation-platform/automation-portal:2.2!ansible-plugin-backstage-rhaap'
    disabled: false
    pluginConfig:
      dynamicPlugins:
        frontend:
          ansible.plugin-backstage-rhaap:
            appIcons:
              - importName: AnsibleLogo
                name: AnsibleLogo
            dynamicRoutes:
              - importName: AnsiblePage
                menuItem:
                  icon: AnsibleLogo
                  text: Ansible
                path: /ansible

  # Self-service field extensions (AAP token picker, resource picker)
  - package: 'oci://registry.redhat.io/ansible-automation-platform/automation-portal:2.2!ansible-plugin-backstage-self-service'
    disabled: false
    pluginConfig:
      dynamicPlugins:
        frontend:
          ansible.plugin-backstage-self-service:
            scaffolderFieldExtensions:
              - importName: AAPTokenFieldExtension
              - importName: AAPResourcePickerExtension

  # Catalog backend module
  - package: 'oci://registry.redhat.io/ansible-automation-platform/automation-portal:2.2!ansible-backstage-plugin-catalog-backend-module-rhaap'
    disabled: false

  # Auth backend (optional — disable if using OIDC)
  - package: 'oci://registry.redhat.io/ansible-automation-platform/automation-portal:2.2!ansible-backstage-plugin-auth-backend-module-rhaap-provider'
    disabled: true

  # Scaffolder actions (launch job templates, etc.)
  - package: 'oci://registry.redhat.io/ansible-automation-platform/automation-portal:2.2!ansible-plugin-scaffolder-backend-module-backstage-rhaap'
    disabled: false
    pluginConfig:
      dynamicPlugins:
        backend:
          ansible.plugin-scaffolder-backend-module-backstage-rhaap:
```

Note: These come from `registry.redhat.io`, not `ghcr.io`.

## App-Config

```yaml
ansible:
  creatorService:
    baseUrl: 127.0.0.1
    port: '8000'
  devSpaces:
    baseUrl: https://devspaces.apps.<cluster-domain>
  rhaap:
    baseUrl: ${AAP_BASE_URL}
    token: ${AAP_TOKEN}
    checkSSL: false

scaffolder:
  defaultEnvironment:
    secrets:
      AAP_TOKEN: ${AAP_TOKEN}
      aapToken: ${AAP_TOKEN}
      token: ${AAP_TOKEN}
```

## Secrets

AAP credentials synced via ESO (or stored as OpenShift Secrets):

```yaml
# aap-credentials-eso.yaml
AAP_BASE_URL: https://aap.example.com
AAP_TOKEN: <controller-api-token>
```

## Using AAP in Software Templates

Templates can reference the AAP token via environment secrets:

```yaml
${{ environment.secrets.AAP_TOKEN }}
```

This is RHDH syntax — NOT upstream Backstage's `${{ secrets.* }}`.

## Using AAP in Orchestrator Workflows

Orchestrator workflows call AAP via OpenAPI functions (not custom REST). See [orchestrator/sonataflow-workflows.md](../orchestrator/sonataflow-workflows.md) for the pattern.

The namespace-approval workflow demonstrates this: it uses an OpenAPI spec for the AAP Controller API to launch job templates and poll for completion.
