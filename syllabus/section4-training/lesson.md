# Section 4: Training with Kubeflow

## Learning Objectives
- Run ML training jobs on Kubernetes
- Understand distributed training concepts
- Use PyTorch/TensorFlow operators
- Monitor training progress

---

## Lesson 4.1: Training Operators Overview

### What are Training Operators?

Kubeflow provides **Kubernetes operators** for distributed training:

| Operator | Framework | Use Case |
|----------|-----------|----------|
| **TFJob** | TensorFlow | Distributed TensorFlow |
| **PyTorchJob** | PyTorch | Distributed PyTorch |
| **MPIJob** | Horovod/MXNet | Multi-node training |
| **XGBoostJob** | XGBoost | Gradient boosting |

### Architecture

```
┌─────────────────────────────────────────────────────┐
│                  Training Job                       │
├─────────────────────────────────────────────────────┤
│  Master (coordination, checkpointing)              │
├─────────────────────────────────────────────────────┤
│  Worker 1    │    Worker 2    │    Worker N        │
│  (training)  │    (training)  │    (training)      │
└─────────────────────────────────────────────────────┘
```

---

## Lesson 4.2: Single-Node Training

### PyTorchJob Example

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
            image: pytorch/pytorch:latest
            command:
              - python
              - /opt/train.py
            resources:
              limits:
                memory: "4Gi"
                cpu: "2"
```

### Create Training Job

```bash
kubectl apply -f pytorch-job.yaml
```

### Monitor Training

```bash
# Check job status
kubectl get pytorchjobs -n kubeflow

# Watch pod logs
kubectl logs -f pytorch-training-master-0 -n kubeflow
```

---

## Lesson 4.3: Distributed Training

### Distributed PyTorchJob

```yaml
apiVersion: kubeflow.org/v1
kind: PyTorchJob
metadata:
  name: distributed-pytorch
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
            image: your-training-image
            command:
              - python
              - /opt/train.py
            env:
              - name: WORLD_SIZE
                value: "3"
              - name: MASTER_ADDR
                value: "pytorch-training-master-0"
    Worker:
      replicas: 2
      restartPolicy: OnFailure
      template:
        spec:
          containers:
          - name: pytorch
            image: your-training-image
            command:
              - python
              - /opt/train.py
            env:
              - name: WORLD_SIZE
                value: "3"
              - name: MASTER_ADDR
                value: "pytorch-training-master-0"
```

### Key Environment Variables

| Variable | Description |
|----------|-------------|
| WORLD_SIZE | Total number of workers |
| RANK | Current worker rank (0 = master) |
| MASTER_ADDR | Master pod address |
| MASTER_PORT | Port for communication |

---

## Lesson 4.4: TensorFlow Distributed Training

### TFJob Example

```yaml
apiVersion: kubeflow.org/v1
kind: TFJob
metadata:
  name: distributed-tf
  namespace: kubeflow
spec:
  tfReplicaSpecs:
    Chief:
      replicas: 1
      restartPolicy: OnFailure
      template:
        spec:
          containers:
          - name: tensorflow
            image: tensorflow/tensorflow:latest
            command:
              - python
              - /opt/train.py
    Worker:
      replicas: 2
      restartPolicy: OnFailure
      template:
        spec:
          containers:
          - name: tensorflow
            image: tensorflow/tensorflow:latest
            command:
              - python
              - /opt/train.py
    PS:
      replicas: 1
      restartPolicy: OnFailure
      template:
        spec:
          containers:
          - name: tensorflow
            image: tensorflow/tensorflow:latest
            command:
              - python
              - /opt/train.py
```

### TensorFlow Roles

| Role | Purpose |
|------|---------|
| **Chief** | Coordinates training, manages checkpoints |
| **Worker** | Performs training computation |
| **PS (Parameter Server)** | Stores model parameters (for older TF) |

---

## Lesson 4.5: Monitoring Training

### Check Job Status

```bash
# List all training jobs
kubectl get trainingjobs -n kubeflow

# Get detailed status
kubectl describe pytorchjob distributed-pytorch -n kubeflow
```

### View Logs

```bash
# Master pod logs
kubectl logs pytorch-training-master-0 -n kubeflow

# Worker pod logs
kubectl logs pytorch-training-worker-0 -n kubeflow
```

### Using TensorBoard

```bash
# Port forward to TensorBoard
kubectl port-forward svc/tensorboard -n kubeflow 6006:80

# Open http://localhost:6006
```

### Save Model to MinIO

```python
import minio
from minio import Minio

client = Minio("minio.kubeflow:9000", access_key="minioadmin", secret_key="minioadmin")
client.fput_object("models", "classifier.pt", "/tmp/model.pt")
```

---

## Key Takeaways

1. **Training Operators** = Kubeflow CRDs for ML training
2. **PyTorchJob/TFJob** = Run distributed training on K8s
3. **Environment variables** = Configure worker communication
4. **Monitoring** = Check status with kubectl, view logs, use TensorBoard

---

## Next Section

Section 5: Hyperparameter Tuning with Katib
- Define search spaces for HPO
- Run Katib experiments
- Analyze results
