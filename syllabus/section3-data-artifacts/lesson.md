# Section 3: Data & Artifact Management

## Learning Objectives
- Understand Kubeflow's artifact system
- Set up MinIO for local storage
- Pass data between pipeline components
- Track metadata and lineage

---

## Lesson 3.1: Understanding Artifacts

### What are Artifacts?

Artifacts are **outputs and inputs** to pipeline components. They represent data objects like datasets, models, and metrics.

### Artifact Types

| Type | Description | Example |
|------|-------------|---------|
| **Dataset** | Input/output data | training_data.csv |
| **Model** | Trained model file | model.pt |
| **Metrics** | Evaluation results | accuracy.json |
| **Visualization** | Charts and plots | roc_curve.png |

### Artifact Metadata

Each artifact stores:
- Name and type
- URI (storage location)
- Created timestamp
- Lineage (which component created it)
- Properties (custom metadata)

---

## Lesson 3.2: MinIO Setup

### What is MinIO?

MinIO is an **S3-compatible object store** used by Kubeflow for storing artifacts locally.

### Install MinIO on Kubernetes

```bash
# Install via manifests
kubectl apply -f https://raw.githubusercontent.com/minio/minio/master/docs/kubernetes/k8d/k8d- deployment.yaml
```

### Port Forward to MinIO Console

```bash
kubectl port-forward svc/minio-console -n kubeflow 9001:9090
```

### Access MinIO
- URL: http://localhost:9001
- Default credentials: minioadmin / minioadmin

### Create Bucket for Artifacts

```bash
# Using mc client
mc alias set myminio http://localhost:9000 minioadmin minioadmin
mc mb myminio/kubeflow-artifacts
```

### Configure KFP to Use MinIO

In Kubeflow Pipelines, configure artifact storage:

```yaml
# pipeline.yaml
metadata:
  annotations:
    kubeflow.org/pipeline-sdk-type: KFP
spec:
  params:
    - name: url
      type: string
      default: s3://kubeflow-artifacts/data
```

---

## Lesson 3.3: Passing Data Between Components

### Input/Output Definitions

```yaml
# component.yaml
inputs:
  - {name: training_data, type: Dataset}
outputs:
  - {name: model, type: Model, description: 'Trained PyTorch model'}
parameters:
  - {name: epochs, type: Integer, default: '10'}
```

### Data Passing Example

```python
@kfp.dsl.pipeline(name='data-pipeline')
def ml_pipeline(url: str):
    # Step 1: Download data
    data = download_op(url=url)
    
    # Step 2: Preprocess (receives data artifact)
    processed = preprocess_op(raw_data=data.output)
    
    # Step 3: Train (receives processed data)
    model = train_op(data=processed.output, epochs=10)
    
    # Step 4: Evaluate
    metrics = evaluate_op(model=model.output, data=processed.output)
```

### Artifact URI Format

```
s3://bucket-name/path/to/artifact
minio://minio-server:9000/bucket/path
```

---

## Lesson 3.4: Metadata Tracking

### Kubeflow Metadata (MLMD)

MLMD tracks:
- Pipeline runs
- Artifact lineage
- Execution history
- Property validation

### Record Metadata in Component

```python
from kubeflow.metadata import metadata

def train_component():
    # Record training metadata
    metadata.Store(
        artifact_type="model",
        name="trained-classifier",
        uri="s3://models/classifier",
        properties={
            "framework": "pytorch",
            "accuracy": 0.95,
        }
    )
```

### Query Metadata

```python
from kubeflow.metadata import metadata

# Get all trained models
models = metadata.Store().get_artifacts(
    type_name="model",
    uri_prefix="s3://models"
)

for model in models:
    print(f"{model.name}: {model.properties['accuracy']}")
```

---

## Lesson 3.5: Volume Mounts for Large Data

### When to Use Volumes

For datasets >100MB, use volume mounts instead of artifact passing.

### PVC Example

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 10Gi
```

### Mount in Component

```yaml
implementation:
  container:
    image: pytorch/pytorch
    command: ["python", "train.py"]
    volumeMounts:
      - name: data
        mountPath: /data
    volumes:
      - name: data
        persistentVolumeClaim:
          claimName: data-pvc
```

---

## Key Takeaways

1. **Artifacts** = data objects passed between components
2. **MinIO** = S3-compatible storage (local)
3. **URI format** = s3://bucket/path for artifact locations
4. **Metadata** = tracks lineage and properties
5. **Volumes** = for large data (>100MB)

---

## Next Section

Section 4: Training with Kubeflow
- Run training jobs on Kubernetes
- Use PyTorch/TensorFlow operators
- Understand distributed training
