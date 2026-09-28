# OpenShift OAuth as RHDH Auth Provider

## Can You Use It?

Yes. On clusters using the built-in OpenShift OAuth server (OCP < 4.20, or 4.20+ without external OIDC enabled), RHDH can authenticate against it. This is one of the simplest setups since RHDH is already on the cluster.

## Considerations

- **OCP 4.20+ external OIDC**: If the cluster uses external OIDC (which replaces the built-in OAuth server), point RHDH at the external OIDC provider directly instead.
- **User/group sync**: The built-in OAuth authenticates users, but you still need a catalog provider to populate User/Group entities in RHDH for RBAC. OpenShift groups are not automatically discovered by RHDH.
- **Simpler alternative**: If the customer already has Okta/Keycloak/AD, pointing RHDH directly at that OIDC provider is usually cleaner than going through OpenShift OAuth as a middleman.

## When to Recommend

- Quick PoC or demo environments
- Clusters where OpenShift OAuth is already the primary auth mechanism and no external IdP is in play
- Situations where adding Keycloak or configuring direct OIDC to an external IdP is out of scope

## When NOT to Recommend

- Production environments with an external IdP (Okta, Entra ID, etc.) — go direct
- Clusters on OCP 4.20+ using external OIDC — the built-in OAuth server isn't running
- When you need user/group sync — you'll still need a separate catalog provider anyway
