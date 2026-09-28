# ArgoCD OIDC Configuration (OpenShift GitOps)

## Approach

Use `spec.oidcConfig` on the ArgoCD CR for direct OIDC (Keycloak).

Do **NOT** use `spec.sso` or `spec.extraConfig["oidc.config"]`.

## Why

- `spec.sso` and `spec.oidcConfig` are mutually exclusive — the CRD validation rejects both.
- `spec.extraConfig["oidc.config"]` works as a workaround but the operator strips `rootCA` during reconciliation.
- `spec.oidcConfig` is the documented, operator-managed approach that properly propagates all fields including `rootCA`.

## Key Points

- Use `spec.oidcConfig` (string field) with the full OIDC YAML config including `rootCA` for self-signed CAs
- Remove `spec.sso` entirely (don't set it to `"none"` — that's not a valid value)
- Keep `spec.extraConfig` for non-OIDC settings (accounts, etc.)
- After OIDC config changes, restart the server pod and clear browser cookies

## Docs

- [OpenShift GitOps 1.21 — Access Control](https://docs.redhat.com/en/documentation/red_hat_openshift_gitops/1.21/html-single/access_control_and_user_management/index)
