# OpenShift External OIDC Authentication

OpenShift external OIDC authentication (GA in OCP 4.20+, Tech Preview in 4.19) replaces the built-in OAuth server with direct kube-apiserver OIDC integration. This is the standardized Kubernetes-native approach.

## Key Docs

Use `dev-browser` to read these (JS-rendered pages that WebFetch can't handle):

- [OCP 4.20 external auth](https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html/authentication_and_authorization/external-auth)
- [Red Hat Architect reference architecture](https://www.redhat.com/architect/portfolio/detail/134-openshift-external-auth/index.html)
- [Keycloak config for external auth](https://www.redhat.com/architect/portfolio/detail/134-openshift-external-auth/05-keycloak.html)
- [Enhancement proposal](https://github.com/openshift/enhancements/blob/master/enhancements/authentication/direct-external-oidc-provider.md)

## Notes

The reference architecture uses Red Hat Build of Keycloak (RHBK) as the reference IdP but any OIDC provider works.
