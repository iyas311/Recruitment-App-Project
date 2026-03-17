# Kubernetes files

Apply in this order:

```bash
kubectl apply -f k8s/mongodb-secret.sample.yaml
kubectl apply -f k8s/mongodb-service.yaml
kubectl apply -f k8s/mongodb-statefulset.yaml
kubectl apply -f k8s/hr-app-configmap.yaml
kubectl apply -f k8s/hr-app-secret.sample.yaml
kubectl apply -f k8s/hr-app-deployment.yaml
kubectl apply -f k8s/hr-app-service.yaml
```

## Important

- Default sample secrets are prefilled so manifests can run quickly; rotate them before production.
- Keep `k8s/hr-app-secret.sample.yaml` and `k8s/mongodb-secret.sample.yaml` passwords in sync.
- Default image is `hr-app:latest`; change it if your image name/registry is different.
- Optional hardening: create a dedicated Mongo app user (`hrapp`) and then update `MONGO_URI` to use that user.
