# Demo: MinIO Setup for Kubeflow

## Overview
Set up MinIO for local artifact storage in Kubeflow.

## Time
10 minutes

## Steps

### 1. Install MinIO Operator
```bash
kubectl apply -f https://raw.githubusercontent.com/minio/operator/master/kubernetes.yaml
```

### 2. Create MinIO Instance
```yaml
apiVersion: minio.min.io/v4
kind: Tenant
metadata:
  name: minio
  namespace: kubeflow
spec:
  pools:
    - servers: 1
      volumesPerServer: 4
      resources:
        requests:
          cpu: 250m
          memory: 1Gi
```

```bash
kubectl apply -f minio-tenant.yaml
```

### 3. Port Forward to Console
```bash
kubectl port-forward svc/minio-console -n kubeflow 9001:9090
```

### 4. Access MinIO Console
- URL: http://localhost:9001
- Access Key: minioadmin
- Secret Key: minioadmin

### 5. Create Bucket
```bash
# Using mc client
mc alias set myminio http://localhost:9000 minioadmin minioadmin
mc mb myminio/kubeflow-artifacts -p
```

## Verify Setup
```bash
# List buckets
mc ls myminio
```

## Key Takeaways
- MinIO provides S3-compatible storage
- Artifacts stored locally, no cloud needed
- Configure Kubeflow to use MinIO for pipeline artifacts
