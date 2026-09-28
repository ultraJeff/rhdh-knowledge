# RHDH Documentation and Research Notes

## Always Use Official Docs

When configuring RHDH, always verify against official Red Hat docs rather than community sites, research agent output, or older example repos.

The orchestrator plugin is actively evolving and permission names, config keys, and plugin package names change between versions.

## Key Doc URLs

- Authentication: `https://docs.redhat.com/en/documentation/red_hat_developer_hub/<version>/html-single/authentication_in_red_hat_developer_hub/index`
- Authorization/RBAC: `https://docs.redhat.com/en/documentation/red_hat_developer_hub/<version>/html-single/authorization_in_red_hat_developer_hub/index`
- Plugin configuration: `https://docs.redhat.com/en/documentation/red_hat_developer_hub/<version>/html-single/configuring_plugins_in_red_hat_developer_hub/index`
- Orchestrator: `https://docs.redhat.com/en/documentation/red_hat_developer_hub/<version>/html-single/orchestrator_in_red_hat_developer_hub/index`

Use `dev-browser` to fetch Red Hat docs pages that return 403 via WebFetch (JS-rendered).

## Community Resources

- Orchestrator community: `https://rhdhorchestrator.io` — useful workflow examples but may lag behind official plugin
- Orchestrator Helm repo: `https://rhdhorchestrator.io/serverless-workflows`
- RHDH plugin export overlays: `ghcr.io/redhat-developer/rhdh-plugin-export-overlays/`

## Reference Architectures

- [RHDH Okta Auth](https://github.com/redhat-na-ssa/rhdh-okta-auth) — Keycloak federation with Okta
- [Backstage Auth/Catalog Providers Guide](https://developers.redhat.com/articles/2025/01/07/backstage-authentication-and-catalog-providers-practical-guide)
