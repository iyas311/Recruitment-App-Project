# Kubernetes env key/value samples

Use the following values as a working sample for this project.

## ConfigMap (`hr-app-config`)

```yaml
data:
  PORT: "3000"
  DB_NAME: "hr_system"
  MONGO_HOST: "mongodb"
  MONGO_PORT: "27017"
```

## Secret (`hr-app-secrets`)

```yaml
stringData:
  MONGO_APP_USERNAME: "hrapp"
  MONGO_APP_PASSWORD: "ChangeMe_hrapp_123"
  MONGO_ROOT_USERNAME: "root"
  MONGO_ROOT_PASSWORD: "ChangeMe_root_123"
  MONGO_URI: "mongodb://hrapp:ChangeMe_hrapp_123@mongodb:27017/hr_system?authSource=hr_system"
```

## How to wire these values

### `hr-app` container env

- `PORT` from ConfigMap
- `DB_NAME` from ConfigMap
- `MONGO_URI` from Secret

### `mongodb` container env

- `MONGO_INITDB_ROOT_USERNAME` = Secret `MONGO_ROOT_USERNAME`
- `MONGO_INITDB_ROOT_PASSWORD` = Secret `MONGO_ROOT_PASSWORD`
- `MONGO_INITDB_DATABASE` = ConfigMap `DB_NAME`

## Important

- `MONGO_URI` and `DB_NAME` are the variables consumed directly by `server.js`.
- The init scripts currently create an application DB user named `hrapp` in database `hr_system`.
- Replace all sample passwords with strong values before production use.
