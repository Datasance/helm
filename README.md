# Datasance PoT Helm Chart

This chart deploys the Datasance PoT operator (iofog-operator) and creates one `ControlPlane` custom resource instance automatically.

## Prerequisites
- Helm 3
- Kubernetes 1.22+ (uses apiextensions.k8s.io/v1 CRDs)

## Add the Helm repo
```
helm repo add datasance https://datasance.github.io/helm
helm repo update
```

## Install
```
helm install pot-operator datasance/pot -n pot --create-namespace \
  -f myvalues.yaml
```
The install creates:
- CRDs for `controlplanes.datasance.com`
- The PoT operator (iofo-operator) Deployment + RBAC + ServiceAccount
- One ControlPlane instance (toggle with `controlplane.create`)

## Configuration
Edit `values.yaml` or provide your own overrides. Key sections:
- `operator.*`: image, replicaCount, resources, scheduling, extraEnv/args, serviceAccount and RBAC toggles.
- `controlplane.*`: metadata and full `.spec` for the ControlPlane CR (auth, database, controller, events, images, services, ingresses, replicas).
- `crds.install`: control CRD installation.

Required values (must set via `values.yaml` or `--set`):
- `controlplane.spec.auth.url`
- `controlplane.spec.auth.realm`
- `controlplane.spec.auth.realmKey`
- `controlplane.spec.auth.ssl`
- `controlplane.spec.auth.controllerClient`
- `controlplane.spec.auth.controllerSecret`
- `controlplane.spec.auth.viewerClient`

Common `--set` flags:
- Operator:
  - `operator.image=ghcr.io/datasance/operator:3.5.4`
  - `operator.replicaCount=1`
  - `operator.imagePullPolicy=Always`
  - `operator.resources.{requests,limits}.cpu|memory=<value>`
  - `operator.nodeSelector.<key>=<value>`
  - `operator.tolerations[0].key=<key>` (with .operator, .effect, .value as needed)
  - `operator.affinity.nodeAffinity` / `operator.affinity.podAntiAffinity` (YAML via `--set-string`)
  - `operator.serviceAccount.create=true|false`
  - `operator.serviceAccount.name=<sa-name>` (when create=false to use existing)
  - `operator.imagePullSecrets[0].name=<secret>`
- ControlPlane metadata:
  - `controlplane.create=true|false`
  - `controlplane.name=<name>`
  - `controlplane.namespace=<ns>` (defaults to release namespace)
- ControlPlane auth (required):
  - `controlplane.spec.auth.url=<url>`
  - `controlplane.spec.auth.realm=<realm>`
  - `controlplane.spec.auth.realmKey=<realmKey>`
  - `controlplane.spec.auth.ssl=true|false`
  - `controlplane.spec.auth.controllerClient=<client>`
  - `controlplane.spec.auth.controllerSecret=<secret>`
  - `controlplane.spec.auth.viewerClient=<client>`
- ControlPlane database (optional; omit for SQLite and keep controller replicas=1):
  - `controlplane.spec.database.provider=<postgres|...>`
  - `controlplane.spec.database.host=<db-host>`
  - `controlplane.spec.database.port=<port>`
  - `controlplane.spec.database.user=<user>`
  - `controlplane.spec.database.password=<password>`
  - `controlplane.spec.database.databaseName=<name>`
  - `controlplane.spec.database.ssl=true|false`
  - `controlplane.spec.database.ca=<base64-CA>`
- ControlPlane runtime:
  - `controlplane.spec.replicas.controller=<int>`
  - `controlplane.spec.controller.logLevel=<level>`
  - `controlplane.spec.controller.https=true|false`
  - `controlplane.spec.controller.secretName=<tls-secret>`
- ControlPlane ingress/services:
  - `controlplane.spec.ingresses.controller.host=<host>`
  - `controlplane.spec.ingresses.controller.ingressClassName=<class>`
  - `controlplane.spec.ingresses.controller.annotations.<key>=<value>`
  - `controlplane.spec.services.controller.type=<LoadBalancer|ClusterIP|NodePort>`
  - `controlplane.spec.services.controller.annotations.<key>=<value>`
  - Router ingress/service: `controlplane.spec.ingresses.router.*`, `controlplane.spec.services.router.*`
- ControlPlane images:
  - `controlplane.spec.images.pullSecret=<secret>`
  - `controlplane.spec.images.controller=<image>`
  - `controlplane.spec.images.router=<image>`
- ControlPlane events:
  - `controlplane.spec.events.auditEnabled=true|false`
  - `controlplane.spec.events.retentionDays=<int>`
  - `controlplane.spec.events.cleanupInterval=<int>`
  - `controlplane.spec.events.captureIpAddress=true|false`

Quick install with flags (example):
```
helm install pot datasance/pot -n pot --create-namespace \
  --set controlplane.spec.auth.url=https://keycloak.example.com \
  --set controlplane.spec.auth.realm=pot \
  --set controlplane.spec.auth.realmKey=master \
  --set controlplane.spec.auth.ssl=true \
  --set controlplane.spec.auth.controllerClient=pot-controller \
  --set controlplane.spec.auth.controllerSecret=supersecret \
  --set controlplane.spec.auth.viewerClient=pot-viewer
```

Example override:
```yaml
operator:
  image: ghcr.io/datasance/operator:3.5.4
controlplane:
  spec:
    # Database is optional; if omitted, the controller uses internal SQLite.
    # When using SQLite, keep controller replicas at 1.
    database:
      provider: postgres
      host: db
      port: 5432
      user: pot
      password: changeme
      databaseName: pot
      ssl: false
    auth:
      url: https://keycloak.example.com
      realm: pot
      realmKey: master
      ssl: true
      controllerClient: pot-controller
      controllerSecret: supersecret
      viewerClient: pot-viewer
```

## Upgrade
```
helm upgrade pot-operator datasance/pot -n pot -f myvalues.yaml
```

## Uninstall
```
helm uninstall pot -n pot
```
CRDs remain by default; remove manually if desired.
