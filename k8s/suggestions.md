# Kubernetes suggestions for this repo

I re-checked the repository and aligned these suggestions with current app/runtime behavior.

## 1) Critical values to verify first

- App **must** receive:
  - `PORT` (ConfigMap)
  - `DB_NAME` (ConfigMap)
  - `MONGO_URI` (Secret)
- For root Mongo authentication, use `authSource=admin` in connection strings.
- If you use an app-scoped Mongo user (recommended), use `authSource=hr_system`.

## 2) Recommended ConfigMap and Secret keys

### ConfigMap (`hr-app-config`)

```yaml
data:
  PORT: "3000"
  DB_NAME: "hr_system"
```

### Secret (`hr-app-secrets`)

```yaml
stringData:
  # Option A: recommended app user URI
  MONGO_URI: "mongodb://hrapp:<APP_PASSWORD>@mongodb:27017/hr_system?authSource=hr_system"

  # Option B: root URI (only if needed)
  MONGO_ROOT_URI: "mongodb://<ROOT_USER>:<ROOT_PASSWORD>@mongodb:27017/hr_system?authSource=admin"

  # Mongo bootstrap credentials
  MONGO_ROOT_USERNAME: "<ROOT_USER>"
  MONGO_ROOT_PASSWORD: "<ROOT_PASSWORD>"
```

## 3) Workload suggestions

### `hr-app` Deployment

- `containerPort: 3000`
- Env wiring:
  - `PORT` <- ConfigMap `PORT`
  - `DB_NAME` <- ConfigMap `DB_NAME`
  - `MONGO_URI` <- Secret `MONGO_URI`
- Add probes:
  - `livenessProbe`: GET `/` on `3000`
  - `readinessProbe`: GET `/` on `3000`
- Set baseline resources:
  - requests: `cpu: 100m`, `memory: 128Mi`
  - limits: `cpu: 500m`, `memory: 512Mi`
- Use `replicas: 2` minimum.

### `mongodb` (prefer StatefulSet + PVC)

- Prefer StatefulSet over Deployment for persistent DB data.
- Env wiring:
  - `MONGO_INITDB_ROOT_USERNAME` <- Secret `MONGO_ROOT_USERNAME`
  - `MONGO_INITDB_ROOT_PASSWORD` <- Secret `MONGO_ROOT_PASSWORD`
  - `MONGO_INITDB_DATABASE` <- ConfigMap `DB_NAME`
- Service: headless Service (`clusterIP: None`) named `mongodb` on `27017` for StatefulSet DNS.
- Attach persistent volume (for example, `10Gi`).

## 4) Service and access

- `hr-app` Service:
  - `ClusterIP` for internal-only or with Ingress in front
  - `LoadBalancer` only when direct external exposure is required
  - Service port `80` -> targetPort `3000`
- `mongodb` Service:
  - `ClusterIP`, port `27017`
  - do not expose publicly unless absolutely required

## 5) Security and operations

- Keep only non-sensitive values in ConfigMap; put credentials/URIs in Secret.
- Add app `securityContext` where compatible:
  - `runAsNonRoot: true`
  - `allowPrivilegeEscalation: false`
  - `readOnlyRootFilesystem: true`
- Add a PodDisruptionBudget for app replicas.
- Optionally add HPA for `hr-app`.

## 6) Repo-specific alignment notes

- App consumes only `PORT`, `MONGO_URI`, and `DB_NAME`.
- App listens on `0.0.0.0`, so Kubernetes Service routing works.
- Mongo init scripts create/use DB `hr_system` and app user `hrapp`.
