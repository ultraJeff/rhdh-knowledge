# Keycloak Setup for RHDH

Scope: Keycloak configuration decisions that affect RHDH specifically. Not a general Keycloak admin guide.

## Realm Design

Use a dedicated realm for RHDH (e.g., `rhdh`), separate from the realm used for OpenShift cluster auth. This avoids cross-contamination of client scopes and realm settings.

Example from tallgeese:
- `openshift` realm — cluster OAuth integration
- `rhdh` realm — RHDH OIDC authentication

## Client Configuration

Create an OIDC client in the RHDH realm:
- Client type: OpenID Connect
- Client authentication: On (confidential)
- Valid redirect URIs: `https://<rhdh-url>/api/auth/oidc/handler/frame`
- Web origins: `https://<rhdh-url>`

## Realm Import Gotcha

When importing a realm via `--import-realm`, if the JSON includes a `clientScopes` array, Keycloak treats it as the complete list and does NOT auto-create built-in scopes (profile, email, etc.).

If `clientScopes` is omitted, Keycloak creates all built-ins automatically.

**Impact on RHDH:** Missing `profile` scope means `preferred_username` is absent from tokens, causing identity resolution to fail or fall back to the Keycloak `sub` UUID.

**Fix:** Always include built-in scope definitions explicitly in the `clientScopes` array when defining custom scopes.

## Federating External IdPs

Keycloak can federate Okta, Entra ID, or other OIDC providers via identity brokering. This is the recommended architecture when RHDH needs user/group sync from an external IdP — the `keycloakOrg` catalog provider (preinstalled in RHDH) then syncs the federated users/groups.

See [okta.md](../catalog-providers/okta.md) for the Okta federation pattern.

## Secrets

RHDH needs these Keycloak values as secrets:
- `KEYCLOAK_BASE_URL` — e.g., `https://sso.example.com`
- `KEYCLOAK_REALM` — e.g., `rhdh`
- `KEYCLOAK_CLIENT_ID`
- `KEYCLOAK_CLIENT_SECRET`

Recommended: sync via External Secrets Operator from Vault or another secrets manager.
