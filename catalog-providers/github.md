# GitHub Catalog Provider

## Integration Config

```yaml
integrations:
  github:
    - host: ${GITHUB_INTEGRATION_HOST_DOMAIN}
      token: ${GITHUB_PAT}
      apps:
        - appId: ${GITHUB_INTEGRATION_APP_ID}
          clientId: ${GITHUB_INTEGRATION_CLIENT_ID}
          clientSecret: ${GITHUB_INTEGRATION_CLIENT_SECRET}
          privateKey: |
            ${GITHUB_INTEGRATION_PRIVATE_KEY_FILE}
```

## Discovery Plugin

```yaml
plugins:
  - package: ./dynamic-plugins/dist/backstage-plugin-catalog-backend-module-github-dynamic
    disabled: false
```

```yaml
catalog:
  providers:
    github:
      providerId:
        organization: "${GITHUB_INTEGRATION_ORGANIZATION}"
        schedule:
          frequency:
            minutes: 30
          initialDelay:
            seconds: 15
          timeout:
            minutes: 15
```

## GitHub Org Provider (User/Group Sync)

```yaml
plugins:
  - package: ./dynamic-plugins/dist/backstage-plugin-catalog-backend-module-github-org-dynamic
    disabled: false
```

```yaml
catalog:
  providers:
    githubOrg:
      id: githubOrg
      githubUrl: "https://github.com"
      orgs:
        - ${GITHUB_INTEGRATION_ORGANIZATION}
      schedule:
        frequency:
          minutes: 30
        initialDelay:
          seconds: 15
        timeout:
          minutes: 15
```

## Scaffolder Actions

```yaml
plugins:
  - package: ./dynamic-plugins/dist/backstage-plugin-scaffolder-backend-module-github-dynamic
    disabled: false
```

## Backend Reading Allow List

```yaml
backend:
  reading:
    allow:
      - host: raw.githubusercontent.com
      - host: github.com
```
