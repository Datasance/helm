# Datasance PoT Helm Chart

This chart deploys the Datasance PoT operator (iofog-operator) and optionally creates one `ControlPlane` custom resource instance. It is aligned with [iofog-operator](https://github.com/Datasance/iofog-operator) 3.7.2 (ControlPlane CRD v3, NATS, vault).

## Prerequisites

- Helm 3
- Kubernetes 1.22+ (uses apiextensions.k8s.io/v1 CRDs)

## Add the Helm repo

```bash
helm repo add datasance https://datasance.github.io/helm
helm repo update
```

## Install from repo

```bash
helm install pot datasance/pot -n pot --create-namespace -f myvalues.yaml
```

## Install from local chart (e.g. for development)

From the repository root, use the **full values filename** (e.g. `test-values.yaml`) so Helm loads your overrides:

```bash
helm upgrade --install pot ./charts/pot -n pot --create-namespace -f test-values.yaml
```

Using `-f test` looks for a file named `test`, not `test-values.yaml`. Optional fields like `externalTrafficPolicy` are defined in the chart defaults so your overrides (e.g. `Cluster`) merge in correctly.

Or pass the chart defaults first, then your overrides:

```bash
helm upgrade --install pot ./charts/pot -n pot --create-namespace -f charts/pot/values.yaml -f test-values.yaml
```

The install creates:

- CRDs for `controlplanes.datasance.com`
- The PoT operator (iofog-operator) Deployment, RBAC (Role, ClusterRole, bindings), and ServiceAccount
- Optionally one ControlPlane instance (toggle with `controlplane.create`)

## Configuration

Edit `values.yaml` or provide your own overrides (e.g. `test-values.yaml`). Key sections:

- **`operator.*`**: image, replicaCount, resources, scheduling, extraEnv/extraArgs, serviceAccount, RBAC.
- **`controlplane.*`**: metadata and full ControlPlane `.spec` (auth, database, controller, events, images, services, nats, ingresses, vault, replicas).
- **`crds.install`**: whether to install the ControlPlane CRD.

### Required values

Must be set via values or `--set`:

- **Auth** (Keycloak / OIDC): `controlplane.spec.auth.url`, `realm`, `realmKey`, `ssl`, `controllerClient`, `controllerSecret`, `viewerClient`
- **Database** (required in ControlPlane CRD v3): `controlplane.spec.database` with at least `provider`, `host`, `port`, `user`, `password`, `databaseName`. For SQLite use `provider: sqlite` with empty host/port/user/password/databaseName as needed.

### Optional / common overrides

- **Operator**: `operator.image` (default `ghcr.io/datasance/operator:3.7.2`), `operator.replicaCount`, `operator.resources`, `operator.nodeSelector`, `operator.tolerations`, `operator.affinity`, `operator.serviceAccount.create|name`, `operator.imagePullSecrets`
- **ControlPlane metadata**: `controlplane.create`, `controlplane.name`, `controlplane.namespace`
- **Replicas**: `controlplane.spec.replicas.controller`, `controlplane.spec.replicas.nats` (min 2 when NATS enabled)
- **Images**: `controlplane.spec.images.controller`, `router`, `nats`, `pullSecret`
- **Services**: `controlplane.spec.services.controller|router|nats|natsServer` with `type`, `address`, `annotations`, `externalTrafficPolicy` (e.g. `Local` or `Cluster`)
- **NATS**: `controlplane.spec.nats.enabled`, `controlplane.spec.nats.jetStream.memoryStoreSize`, `storageSize`, `storageClassName`
- **Controller**: `controlplane.spec.controller.logLevel`, `https`, `secretName`, `pidBaseDir`, `ecn`, `ecnViewerPort`, `ecnViewerUrl`
- **Ingresses**: `controlplane.spec.ingresses.controller|router|nats` (host, ingressClassName, address, ports)
- **Events**: `controlplane.spec.events.auditEnabled`, `retentionDays`, `cleanupInterval`, `captureIpAddress`
- **Vault** (optional): `controlplane.spec.vault.enabled`, `provider`, `basePath`, and provider-specific config (`hashicorp`, `aws`, `azure`, `google`)

### Quick install with flags (example)

```bash
helm install pot datasance/pot -n pot --create-namespace \
  --set controlplane.spec.auth.url=https://keycloak.example.com \
  --set controlplane.spec.auth.realm=pot \
  --set controlplane.spec.auth.realmKey=master \
  --set controlplane.spec.auth.ssl=external \
  --set controlplane.spec.auth.controllerClient=pot-controller \
  --set controlplane.spec.auth.controllerSecret=supersecret \
  --set controlplane.spec.auth.viewerClient=pot-viewer \
  --set controlplane.spec.database.provider=sqlite \
  --set controlplane.spec.database.host="" \
  --set controlplane.spec.database.port=0 \
  --set controlplane.spec.database.user="" \
  --set controlplane.spec.database.password="" \
  --set controlplane.spec.database.databaseName="" \
  --set controlplane.spec.database.ssl=false
```

### Example values override

```yaml
operator:
  image: ghcr.io/datasance/operator:3.7.2

controlplane:
  create: true
  name: pot
  spec:
    # Database is required in CRD v3. Use sqlite for single-replica or postgres for HA.
    database:
      provider: postgres
      host: db
      port: 5432
      user: pot
      password: changeme
      databaseName: pot
      ssl: false
      ca: ""
    auth:
      url: https://keycloak.example.com
      realm: pot
      realmKey: master
      ssl: external
      controllerClient: pot-controller
      controllerSecret: supersecret
      viewerClient: pot-viewer
    replicas:
      controller: 1
      nats: 2
    images:
      controller: ghcr.io/datasance/controller:3.7.0
      router: ghcr.io/datasance/router:3.7.0
      nats: ghcr.io/datasance/nats:2.12.4
    services:
      controller:
        type: LoadBalancer
      router:
        type: LoadBalancer
      nats:
        type: LoadBalancer
      natsServer:
        type: LoadBalancer
    nats:
      enabled: true
```

## Lint (local chart)

From the repo root:

```bash
helm lint charts/pot
```

## Upgrade

```bash
helm upgrade pot datasance/pot -n pot -f myvalues.yaml
```

For a local chart:

```bash
helm upgrade pot ./charts/pot -n pot -f test-values.yaml
```

## Uninstall

```bash
helm uninstall pot -n pot
```

CRDs are not removed by default; delete them manually if needed.
