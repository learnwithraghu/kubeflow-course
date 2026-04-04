# Section 6: Model Serving with KServe

## Learning Objectives
- Deploy trained models as REST APIs
- Understand InferenceService configuration
- Make predictions and monitor performance
- Configure auto-scaling

---

## Lesson 6.1: Introduction to KServe

### What is KServe?

KServe (formerly KFServing) provides **serverless inference** on Kubernetes.

### Key Features

| Feature | Description |
|---------|-------------|
| **Auto-scaling** | Scale to zero when idle |
| **Canary Deployments** | Test new versions gradually |
| **Model Rollback** | Easy revert to previous version |
| **Multi-model Serving** | Serve multiple models from one pod |

### InferenceService CRD

```yaml
apiVersion: serving.kserve.io/v1
kind: InferenceService
metadata:
  name: classifier
  namespace: kubeflow
spec:
  predictor:
    serviceAccountName: sa
    pytorch:
      storageUri: s3://models/classifier
```

---

## Lesson 6.2: Deploying Models

### Prerequisites
- Trained model saved to storage (MinIO/S3)
- Storage credentials configured

### Deploy PyTorch Model

```yaml
apiVersion: serving.kserve.io/v1
kind: InferenceService
metadata:
  name: mnist-classifier
  namespace: kubeflow
spec:
  predictor:
    pytorch:
      storageUri: s3://kubeflow/models/mnist-classifier
      resources:
        limits:
          cpu: "1"
          memory: 2Gi
```

### Deploy with SKLearn

```yaml
apiVersion: serving.kserve.io/v1
kind: InferenceService
metadata:
  name: sklearn-classifier
  namespace: kubeflow
spec:
  predictor:
    sklearn:
      storageUri: s3://kubeflow/models/sklearn-classifier
```

### Create InferenceService

```bash
kubectl apply -f isvc.yaml
```

---

## Lesson 6.3: Making Predictions

### Get Service Endpoint

```bash
# Get ingress URL
kubectl get ingress -n kubeflow

# Or use port-forward
kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80
```

### REST Inference

```bash
# Get model URL
INGRESS_URL=$(kubectl get ingress -n istio-system -o jsonpath='{.items[0].spec.rules[0].host}')

# Send prediction request
curl -X POST http://${INGRESS_URL}/v1/models/mnist-classifier:predict \
  -H "Content-Type: application/json" \
  -d '{"instances": [[0.1, 0.2, ...]]}'
```

### Request Format

```json
{
  "instances": [
    [0.1, 0.2, 0.3, ...],  // Input 1
    [0.4, 0.5, 0.6, ...]   // Input 2
  ]
}
```

### Response Format

```json
{
  "predictions": [
    {"label": "5", "confidence": 0.95},
    {"label": "2", "confidence": 0.87}
  ]
}
```

---

## Lesson 6.4: Monitoring and Scaling

### Check Service Status

```bash
# List inference services
kubectl get isvc -n kubeflow

# Describe service
kubectl describe isvc mnist-classifier -n kubeflow
```

### Auto-Scaling

KServe uses Knative for auto-scaling:

```yaml
spec:
  predictor:
    minReplicas: 1
    maxReplicas: 10
    autoscaler:
      targetUtilizationPercentage: 70
```

### Scale to Zero

```yaml
spec:
  predictor:
    minReplicas: 0  # Scale to zero when idle
    maxReplicas: 5
```

### View Logs

```bash
# Get predictor pod
kubectl get pods -n kubeflow | grep mnist-classifier

# View logs
kubectl logs mnist-classifier-predictor-xxxx -n kubeflow
```

---

## Lesson 6.5: Canary Deployments

### What is Canary Deployment?

Deploy new version to small percentage of traffic first.

### Canary Configuration

```yaml
apiVersion: serving.kserve.io/v1
kind: InferenceService
metadata:
  name: classifier-canary
  namespace: kubeflow
spec:
  predictor:
    canaryTrafficPercent: 10  # 10% to new version
    sklearn:
      storageUri: s3://kubeflow/models/classifier-v2
```

### Promote Canary

```yaml
# Increase traffic to 100%
kubectl patch isvc classifier --type merge -p '{
  "spec": {"predictor":{"canaryTrafficPercent":100}}
}'
```

---

## Key Takeaways

1. **KServe** = Serverless model inference
2. **InferenceService** = CRD for model deployment
3. **Predictor types** = PyTorch, TensorFlow, SKLearn, etc.
4. **Auto-scaling** = Scale to zero, scale based on traffic
5. **Canary** = Test with small traffic %

---

## Next Section

Section 7: MLOps & Automation
- Connect end-to-end workflows
- Automate pipeline execution
- Apply MLOps best practices
