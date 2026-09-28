# OpenShift Group Sync from Okta

This is an OpenShift platform concern, separate from RHDH user/group catalog sync.

## Option A: OIDC Group Claims via OAuth CR (Supported)

OpenShift syncs group membership at login time based on claims in the ID token.

**Okta-side setup:**
1. Okta admin: Security > API > Authorization Servers > Claims
2. Add `groups` claim to the **ID token** (not Access token)
3. Value type: Groups, Filter: matches desired groups
4. Include `profile` scope

**OAuth CR:**

```yaml
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: okta
    mappingMethod: claim
    type: OpenID
    openID:
      clientID: <client-id>
      clientSecret:
        name: openid-okta-secret
      extraScopes:
      - email
      - profile
      - groups
      claims:
        preferredUsername:
        - preferred_username
        - email
        name:
        - name
        email:
        - email
        groups:
        - groups
      issuer: https://<okta-domain>.okta.com
```

**Gotcha:** Group sync only happens at login time. To force revocation: `oc delete oauthaccesstoken --field-selector=userName="<user>"`

## Option B: Group Sync Operator (Community)

[Group Sync Operator](https://github.com/redhat-cop/group-sync-operator) from Red Hat Community of Practice. **Community-supported, not officially supported.**

```yaml
apiVersion: redhatcop.redhat.io/v1alpha1
kind: GroupSync
metadata:
  name: okta-sync
spec:
  schedule: "*/30 * * * *"
  providers:
    - name: okta
      okta:
        credentialsSecret:
          name: okta-api-token
          namespace: group-sync-operator
        url: "https://<okta-domain>.okta.com/"
        appId: <okta-app-client-id>
```

## Important: OpenShift Groups != RHDH Groups

OpenShift groups do NOT automatically appear in the RHDH catalog. RHDH needs its own catalog provider to import users/groups. See [catalog-providers/okta.md](../catalog-providers/okta.md).

## Docs

- [Configuring Okta as an IdP for OpenShift](https://www.redhat.com/en/blog/how-to-configure-okta-as-an-identity-provider-for-openshift)
- [OIDC Group Claims as alternative to Group Sync Operator](https://access.redhat.com/articles/7116361)
