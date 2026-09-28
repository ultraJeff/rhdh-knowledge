# Keycloak OIDC Authentication in RHDH

## Approach

RHDH uses the generic OIDC auth provider pointing at Keycloak. This is preinstalled and the most common pattern for RHDH on OpenShift. The `keycloakOrg` catalog provider (also preinstalled) can then sync users and groups into the RHDH catalog.

## RHDH App-Config

```yaml
signInPage: oidc

auth:
  session:
    secret: ${BACKEND_SECRET}
  environment: production
  providers:
    oidc:
      production:
        metadataUrl: ${KEYCLOAK_BASE_URL}/realms/${KEYCLOAK_REALM}/.well-known/openid-configuration
        clientId: ${KEYCLOAK_CLIENT_ID}
        clientSecret: ${KEYCLOAK_CLIENT_SECRET}
        prompt: auto
```

## User/Group Sync via keycloakOrg Provider

```yaml
catalog:
  providers:
    keycloakOrg:
      default:
        baseUrl: ${KEYCLOAK_BASE_URL}
        loginRealm: ${KEYCLOAK_REALM}
        realm: ${KEYCLOAK_REALM}
        clientId: ${KEYCLOAK_CLIENT_ID}
        clientSecret: ${KEYCLOAK_CLIENT_SECRET}
        schedule:
          frequency:
            minutes: 5
          timeout:
            minutes: 3
          initialDelay:
            seconds: 15
```

Dynamic plugin (preinstalled, disabled by default):

```yaml
plugins:
  - package: ./dynamic-plugins/dist/backstage-community-plugin-catalog-backend-module-keycloak-dynamic
    disabled: false
```

## Keycloak Realm Import Gotcha

When importing a realm via `--import-realm`, if the realm JSON includes a `clientScopes` array, Keycloak treats it as the complete list and does NOT auto-create built-in scopes (profile, email, etc.). If `clientScopes` is omitted, Keycloak auto-creates all built-ins.

This caused issues where `preferred_username` was missing from tokens because the `profile` scope didn't exist, making OpenShift fall back to the Keycloak `sub` UUID as the username.

**Fix:** Always include built-in scope definitions (`profile`, `email`) explicitly in the `clientScopes` array alongside custom ones.

## Keycloak as Okta Broker

Keycloak can federate Okta via identity brokering. This lets RHDH and OpenShift both authenticate through Keycloak while Okta remains the source of truth. The `keycloakOrg` provider then syncs the federated users/groups into the RHDH catalog.

Reference: [redhat-na-ssa/rhdh-okta-auth](https://github.com/redhat-na-ssa/rhdh-okta-auth)

## RHDH Environment Secrets

RHDH uses `${{ environment.secrets.* }}` — NOT upstream Backstage's `${{ secrets.* }}`.

## Docs

- [RHDH 1.10 — Configuring authentication](https://docs.redhat.com/en/documentation/red_hat_developer_hub/1.10/html-single/authentication_in_red_hat_developer_hub/index)
