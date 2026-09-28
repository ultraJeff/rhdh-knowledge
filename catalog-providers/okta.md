# Syncing Okta Users and Groups into RHDH

For RBAC to work in RHDH, users and groups must exist in the RHDH catalog as Backstage entities. RHDH does not automatically inherit OpenShift groups — it integrates with the same identity providers, so unified RBAC comes from syncing a shared source of truth.

## Option A: Keycloak Federation (Recommended)

Federate Okta through Red Hat Build of Keycloak (RHBK), then use the preinstalled `keycloakOrg` catalog provider. Keycloak acts as an identity broker — Okta remains the source of truth, Keycloak federates, RHDH discovers automatically.

See [keycloak-oidc.md](../authentication/keycloak-oidc.md) for the `keycloakOrg` provider config.

Reference: [redhat-na-ssa/rhdh-okta-auth](https://github.com/redhat-na-ssa/rhdh-okta-auth)

## Option B: Okta Catalog Provider Plugin (Direct)

The `@roadiehq/catalog-backend-module-okta` plugin connects directly to the Okta API. **Not preinstalled** in RHDH — must be loaded as an external dynamic plugin.

```yaml
catalog:
  providers:
    okta:
      default:
        orgUrl: https://<your-okta-domain>.okta.com
        token: ${OKTA_API_TOKEN}
        schedule:
          frequency:
            minutes: 30
          timeout:
            minutes: 10
          initialDelay:
            seconds: 15
        # groupFilter: profile.name sw "rhdh-"
```

Token needs `okta.groups.read` and `okta.users.read` scopes.

## Option C: Static Org Data in Git

Define User and Group entities in YAML and register as a catalog Location.

```yaml
apiVersion: backstage.io/v1alpha1
kind: Group
metadata:
  name: platform-team
spec:
  type: team
  children: []
---
apiVersion: backstage.io/v1alpha1
kind: User
metadata:
  name: jane.doe
spec:
  memberOf:
    - platform-team
```

## Comparison

| | Keycloak (A) | Okta Plugin (B) | Static (C) |
|---|---|---|---|
| Auto-syncs from Okta | Yes (via federation) | Yes (direct API) | No |
| Preinstalled in RHDH | Yes | No | Yes (just YAML) |
| Red Hat supported | Yes | Community | Yes |
| Extra infrastructure | Keycloak instance | None | None |
| Best for | Production, unified identity | Production w/o Keycloak | Small teams, PoC |

## RHDH Supported Catalog Providers for Users/Groups

Five officially supported providers (preinstalled):

1. **Red Hat Build of Keycloak** — `keycloakOrg`
2. **LDAP** — direct LDAP provisioning
3. **GitHub** — `githubOrg`
4. **Microsoft Azure / Entra ID** — `microsoftGraphOrg`
5. **GitLab** — `gitlab` org provider with `orgEnabled: true`

OpenShift is NOT on this list — there is no mechanism where RHDH auto-discovers groups from the OpenShift cluster.
