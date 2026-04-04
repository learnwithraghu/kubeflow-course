# Exercise 4: Run a Training Job

## Objective
Deploy and run a single-node PyTorch training job on Kubernetes.

## Tasks

### Task 1: Create Training Script

**train.py**:
```python
import torch
import torch.nn as nn
import torch.optim as optim
from torchvision import datasets, transforms

print("Starting training...")
print(f"PyTorch version: {torch.__version__}")

# Simple model
model = nn.Sequential(
    nn.Linear(784, 128),
    nn.ReLU(),
    nn.Linear(128, 10)
)

print(f"Model created with {sum(p.numel() for p in model.parameters())} parameters")

# Simulated training
for epoch in range(3):
    print(f"Epoch {epoch+1}/3 - Simulated training...")
    loss = 0.1 / (epoch + 1)
    print(f"Loss: {loss:.4f}")

print("Training complete!")

# Save model
torch.save(model.state_dict(), '/tmp/model.pt')
print("Model saved to /tmp/model.pt")
```

### Task 2: Create Docker Image

**Dockerfile**:
```dockerfile
FROM pytorch/pytorch:latest

WORKDIR /opt
COPY train.py /opt/

CMD ["python", "/opt/train.py"]
```

Build and push:
```bash
docker build -t your-registry/pytorch-train:latest .
docker push your-registry/pytorch-train:latest
```

### Task 3: Create PyTorchJob

**pytorch-job.yaml**:
```yaml
apiVersion: kubeflow.org/v1
kind: PyTorchJob
metadata:
  name: pytorch-training
  namespace: kubeflow
spec:
  pytorchReplicaSpecs:
    Master:
      replicas: 1
      restartPolicy: OnFailure
      template:
        spec:
          containers:
          - name: pytorch
            image: your-registry/pytorch-train:latest
            resources:
              limits:
                memory: "2Gi"
                cpu: "1"
```

### Task 4: Deploy and Monitor

```bash
# Apply job
kubectl apply -f pytorch-job.yaml

# Check status
kubectl get pytorchjobs -n kubeflow

# Watch pods
kubectl get pods -n kubeflow -w

# View logs
kubectl logs pytorch-training-master-0 -n kubeflow
```

## Success Criteria

- [ ] PyTorchJob created successfully
- [ ] Pod runs to completion
- [ ] Logs show training progress
- [ ] Model saved (check logs)

## Cleanup

```bash
kubectl delete -f pytorch-job.yaml -n kubeflow
```

## Challenge

Add more epochs and larger model. Monitor resource usage:
```bash
kubectl top pods -n kubeflow
```
