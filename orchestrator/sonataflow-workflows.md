# SonataFlow Workflow Development

## Profiles

- **dev** — Uses `logic-swf-devmode-rhel9` image, bundles embedded data-index and jobs-service inside the workflow pod. Workflows are NOT visible to the RHDH orchestrator plugin because the embedded data-index is isolated. Good for development only.
- **gitops** — Requires a pre-built container image (`spec.podTemplate.container.image`). Publishes events to the shared platform data-index. The RHDH orchestrator plugin discovers workflows through this shared data-index.

## Building Gitops Images

Use `logic-swf-builder-rhel9:<version>` (match operator version, e.g. 1.37.0 — no `latest` tag). Multi-stage build: builder compiles Quarkus app, runtime uses `ubi9/openjdk-17-runtime`.

## Build-time vs Runtime Properties (CRITICAL)

Many properties that look like they should be runtime-configurable are actually **build-time only**. The operator creates a managed-props ConfigMap (`<name>-managed-props`) but its overrides don't take effect for properties compiled into the Quarkus app. These MUST be baked into `application.properties` at build time:

- `kogito.service.url` — the workflow's own callback URL
- `mp.messaging.outgoing.*.connector` — messaging channel connector type
- `mp.messaging.outgoing.*.url` — messaging channel URLs (jobs-service, data-index)
- `kogito.sw.functions.<name>.host/port` — custom REST function hosts (but DON'T use custom REST)

## Use OpenAPI Functions, Not Custom REST

Custom REST functions (`type: custom, operation: "rest:..."`) use the Vert.x WebClient via `RestWorkItemHandler`. This client has incompatibilities with nginx reverse proxies (400 errors) and doesn't properly handle TLS trust for self-signed certs.

**Use OpenAPI functions instead.** Write a minimal OpenAPI spec, reference it via `workflow-uri-definitions`, and configure with `quarkus.rest-client.<alias>.url`.

```yaml
extensions:
  - extensionid: workflow-uri-definitions
    definitions:
      aap: "specs/aap-controller-openapi.yaml"
functions:
  - name: launchAAPJobTemplate
    operation: aap#launchJobTemplate
```

```properties
quarkus.rest-client.aap.url=https://aap.example.com
quarkus.openapi-generator.aap.auth.BearerToken.bearer-token=${aap_token}
```

## Data-Index Race Condition

After restarts, workflow pods must start AFTER the shared data-index is ready. If they start first, the workflow definition event fails to publish and the workflow won't appear in the orchestrator UI. Fix: restart workflow pods again once data-index is healthy.

## Docs

- Always verify orchestrator config against official RHDH docs, not community sites or older examples
- Official: `https://docs.redhat.com/en/documentation/red_hat_developer_hub/<version>/html-single/orchestrator_in_red_hat_developer_hub/index`
- Community workflows: `https://rhdhorchestrator.io/serverless-workflows` (may lag behind official plugin)
