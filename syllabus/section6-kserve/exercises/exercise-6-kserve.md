# Exercise 6: Deploy Model with KServe

## Objective
Deploy a trained model as an InferenceService and make predictions.

## Prerequisites
- Model saved to MinIO or S3
- KServe installed in Kubeflow

## Tasks

### Task 1: Save Model to MinIO

```python
# Save a simple PyTorch model
import torch
import torch.nn as nn

model = nn.Sequential(
    nn.Linear(10, 5),
    nn.ReLU(),
    nn.Linear(5, 2)
)

# Save locally first
torch.save(model.state_dict(), '/tmp/simple_model.pt')
print("Model saved locally")
```

Upload to MinIO:
```bash
# Install mc client
mc alias set myminio http://localhost:9000 minioadmin minioadmin

# Copy model
mc cp /tmp/simple_model.pt myminio/kubeflow-artifacts/models/
```

### Task 2: Create InferenceService

**isvc.yaml**:
```yaml
apiVersion: serving.kserve.io/v1
kind: InferenceService
metadata:
  name: simple-classifier
  namespace: kubeflow
spec:
  predictor:
    serviceAccountName: sa
    pytorch:
      storageUri: s3://kubeflow-artifacts/models/simple_model.pt
      resources:
        limits:
          cpu: "1"
          memory: 1Gi
```

### Task 3: Deploy

```bash
kubectl apply -f isvc.yaml
```

### Task 4: Check Status

```bash
# Watch until ready
kubectl get isvc -n kubeflow -w

# Check pods
kubectl get pods -n kubeflow | grep simple-classifier
```

### Task 5: Make Prediction

```bash
# Get ingress host
INGRESS_HOST=$(kubectl get ingress istio-ingressgateway -n istio-system -o jsonpath='{.spec.rules[0].host}')

# Port forward if needed
kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80

# Send prediction request
curl -X POST http://localhost:8080/v1/models/simple-classifier:predict \
  -H "Content-Type: application/json" \
  -d '{"instances": [[1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0, 10.0]]}'
```

### Task 6: View Logs

```bash
# Get predictor pod
POD=$(kubectl get pods -n kubeflow -l serving.kserve.io/inferenceservice=simple-classifier -o jsonpath='{.items[0].metadata.name}')

# View logs
kubectl logs $POD -n kubeflow
```

## Success Criteria

- [ ] InferenceService created
- [ ] Service reaches READY state
- [ ] Prediction request succeeds
- [ ] Response returned with predictions

## Cleanup

```bash
kubectl delete -f isvc.yaml -n kubeflow
```

## Challenge

Deploy a second version and use canary traffic splitting:
```yaml
spec:
  predictor:
    canaryTrafficPercent: 20
    pytorch:
      storageUri: s3://models/v2
```
