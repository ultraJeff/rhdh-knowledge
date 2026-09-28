# Namespace Approval Workflow (Worked Example)

A complete example of an orchestrator workflow that provisions an OpenShift namespace via AAP, with notifications at each stage.

## Workflow Definition

`namespace-approval.sw.yaml` — ServerlessWorkflow spec v0.8:

```
Start -> Notify: Request received -> Launch AAP Job -> Wait for Job -> Check Job Status
                                                                          |
                                                         successful -> Notify: Success -> End
                                                         failed    -> Notify: Failed  -> End
                                                         pending   -> Wait for Job (loop)
```

### Key patterns demonstrated:

1. **OpenAPI function definitions** via `workflow-uri-definitions` extension
2. **RHDH notifications** via the shared notifications OpenAPI spec
3. **AAP job template invocation** via a custom OpenAPI spec for AAP Controller
4. **Polling loop** — check job status every 10 seconds until complete
5. **Expression functions** — build structured output using jq-like expressions
6. **Data input/output schemas** — JSON Schema for workflow parameters and results

## Functions

```yaml
extensions:
  - extensionid: workflow-uri-definitions
    definitions:
      notifications: "https://raw.githubusercontent.com/rhdhorchestrator/serverless-workflows/main/workflows/shared/specs/notifications-openapi.yaml"
      aap: "specs/aap-controller-openapi.yaml"
functions:
  - name: createNotification
    operation: notifications#createNotification
  - name: launchAAPJobTemplate
    operation: aap#launchJobTemplate
  - name: getAAPJobStatus
    operation: aap#getJobStatus
  - name: print
    type: custom
    operation: sysout
  - name: successResult
    type: expression
    operation: '{ "result": { "message": "Namespace " + .namespaceName + " provisioned" } }'
```

## Build-Time Properties (application.properties)

These are baked into the container image at build time:

```properties
# Backstage Notifications service
quarkus.rest-client.notifications.url=${BACKSTAGE_NOTIFICATIONS_URL}
quarkus.openapi-generator.notifications.auth.BearerToken.bearer-token=${NOTIFICATIONS_BEARER_TOKEN}

# AAP Controller (OpenAPI)
quarkus.rest-client.aap.url=https://aap.example.com
quarkus.openapi-generator.aap.auth.BearerToken.bearer-token=${aap_token}

# Workflow callback URL
kogito.service.url=http://namespace-approval.sonataflow-infra

# Messaging channels (data-index + jobs-service)
mp.messaging.outgoing.kogito-job-service-job-request-events.connector=quarkus-http
mp.messaging.outgoing.kogito-job-service-job-request-events.url=http://sonataflow-platform-jobs-service.sonataflow-infra/v2/jobs/events
mp.messaging.outgoing.kogito-processdefinitions-events.connector=quarkus-http
mp.messaging.outgoing.kogito-processdefinitions-events.url=http://sonataflow-platform-data-index-service.sonataflow-infra/definitions
mp.messaging.outgoing.kogito-processinstances-events.connector=quarkus-http
mp.messaging.outgoing.kogito-processinstances-events.url=http://sonataflow-platform-data-index-service.sonataflow-infra/processes

quarkus.tls.trust-all=true
kie.flyway.enabled=true
```

## Containerfile (Multi-Stage Build)

```dockerfile
FROM registry.redhat.io/openshift-serverless-1/logic-swf-builder-rhel9:1.37.0 AS builder

COPY namespace-approval.sw.yaml /home/kogito/serverless-workflow-project/src/main/resources/
COPY schemas/ /home/kogito/serverless-workflow-project/src/main/resources/schemas/
COPY specs/ /home/kogito/serverless-workflow-project/src/main/resources/specs/
COPY application.properties /home/kogito/serverless-workflow-project/src/main/resources/

RUN cd /home/kogito/serverless-workflow-project && \
    mvn package -B -ntp -Dquarkus.package.jar.type=mutable-jar -DskipTests

FROM registry.access.redhat.com/ubi9/openjdk-17-runtime:latest

COPY --from=builder --chown=185 /home/kogito/serverless-workflow-project/target/quarkus-app/lib/ /deployments/lib/
COPY --from=builder --chown=185 /home/kogito/serverless-workflow-project/target/quarkus-app/*.jar /deployments/
COPY --from=builder --chown=185 /home/kogito/serverless-workflow-project/target/quarkus-app/app/ /deployments/app/
COPY --from=builder --chown=185 /home/kogito/serverless-workflow-project/target/quarkus-app/quarkus/ /deployments/quarkus/

EXPOSE 8080
USER 185
ENTRYPOINT ["java", "-jar", "/deployments/quarkus-run.jar"]
```

Use `logic-swf-builder-rhel9:<version>` matching your operator version (e.g., 1.37.0). No `latest` tag available.

## Directory Layout

```
namespace-approval/
  namespace-approval.sw.yaml     # Workflow definition
  application.properties         # Build-time Quarkus properties
  Containerfile                  # Multi-stage build
  schemas/
    namespace-approval-schema.json    # Input schema
    workflow-output-schema.json       # Output schema
  specs/
    aap-controller-openapi.yaml  # OpenAPI spec for AAP Controller API
```

## Known Issues

- `job_template_id: 8` is hardcoded — should resolve AAP template by name
- Image currently in registry at `nexus.ultra.lab:5000/sonataflow/namespace-approval:latest`
