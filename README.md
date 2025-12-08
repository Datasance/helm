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
helm install pot datasance/pot -n pot --create-namespace \
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
helm upgrade pot datasance/pot -n pot -f myvalues.yaml
```

## Uninstall
```
helm uninstall pot -n pot
```
CRDs remain by default; remove manually if desired.
