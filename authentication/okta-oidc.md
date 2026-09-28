# Okta OIDC Authentication in RHDH

## Approach

Use the **generic OIDC auth provider** (`auth.providers.oidc`) pointing at Okta. This is preinstalled in RHDH and officially documented — no extra dynamic plugins needed.

There is also a dedicated `auth.providers.okta` provider in upstream Backstage, but it is **not preinstalled** in RHDH and would require loading an additional dynamic plugin (`backstage-plugin-auth-backend-module-okta-provider-dynamic`).

## Okta App Setup

1. In Okta admin console: Applications > Create App Integration > OIDC > **Web**
2. Grant type: **Authorization Code**
3. Sign-in redirect URI: `https://<your-rhdh-url>/api/auth/oidc/handler/frame`
4. Sign-out redirect URI: `https://<your-rhdh-url>`
5. Assign the app to the users/groups who should access RHDH

## RHDH App-Config

```yaml
signInPage: oidc

auth:
  session:
    secret: ${SESSION_SECRET}
  environment: production
  providers:
    oidc:
      production:
        metadataUrl: https://<your-okta-domain>.okta.com/oauth2/default/.well-known/openid-configuration
        clientId: ${OKTA_CLIENT_ID}
        clientSecret: ${OKTA_CLIENT_SECRET}
        prompt: auto
        scope: 'openid profile email groups'
```

## Metadata URL Variants

- Default authorization server: `https://<domain>.okta.com/oauth2/default/.well-known/openid-configuration`
- Custom authorization server: `https://<domain>.okta.com/oauth2/<authServerId>/.well-known/openid-configuration`
- Org-level (no authz server): `https://<domain>.okta.com/.well-known/openid-configuration`

## Gotchas

- `signInPage: oidc` sends users directly to Okta instead of showing a login picker.
- The `session.secret` is **required** for OIDC — without it you get a cryptic "session not found" error.
- **"User not found"**: Users must exist in the RHDH catalog. Without a catalog provider syncing users from Okta, users get "User not found" on login. `dangerouslyAllowSignInWithoutUserInCatalog: true` is a dev-only workaround.

## Docs

- [RHDH 1.10 — Configuring authentication](https://docs.redhat.com/en/documentation/red_hat_developer_hub/1.10/html-single/authentication_in_red_hat_developer_hub/index)
- [RHDH Okta Auth Reference Architecture](https://github.com/redhat-na-ssa/rhdh-okta-auth)
