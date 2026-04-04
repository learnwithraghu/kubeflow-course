# Section 2: Kubeflow Pipelines (KFP)

## Learning Objectives
- Understand pipeline concepts and terminology
- Build a simple 3-step ML pipeline
- Run pipeline and monitor execution
- Modify pipeline parameters

---

## Lesson 2.1: Pipeline Concepts

### What is a Pipeline?

A **Kubeflow Pipeline** is a set of ML tasks connected as a directed acyclic graph (DAG).

### Key Terms

| Term | Definition |
|------|------------|
| **Component** | Single executable task (containerized) |
| **Pipeline** | Collection of components connected in a DAG |
| **Run** | Single execution of a pipeline |
| **Experiment** | Group of related runs |
| **Artifact** | Input/output data (models, datasets, metrics) |

### Pipeline Architecture

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  Component 1 │────▶│  Component 2 │────▶│  Component 3 │
│  (download)  │     │  (preprocess)│     │   (train)    │
└──────────────┘     └──────────────┘     └──────────────┘
       │                    │                     │
       ▼                    ▼                     ▼
   artifact1            artifact2             model.pt
```

### Component Types

1. **Container Component** - Runs a container with specified command
2. **Python Function Component** - Wraps Python function as component

### Simple Example

```python
# Download data component
download_op = kfp.components.load_component_from_file('download/component.yaml')

# Preprocess component
preprocess_op = kfp.components.load_component_from_file('preprocess/component.yaml')

# Train component
train_op = kfp.components.load_component_from_file('train/component.yaml')

# Create pipeline
@kfp.dsl.pipeline(name='my-pipeline', description='Simple ML pipeline')
def my_pipeline(data_url: str, epochs: int):
    data = download_op(url=data_url)
    processed = preprocess_op(data=data.output)
    model = train_op(data=processed.output, epochs=epochs)
```

---

## Lesson 2.2: Building Your First Pipeline

### Prerequisites
- Kubeflow Pipelines SDK installed
- Access to Kubeflow cluster

### Step 1: Install KFP SDK

```bash
pip install kfp
```

### Step 2: Create Component YAML

**download/component.yaml**:
```yaml
name: Download Data
description: Download dataset from URL
outputs:
  - {name: data, type: Dataset}
implementation:
  container:
    image: python:3.9
    command:
      - sh
      - -c
      - |
        wget -O /tmp/data.csv $0
        cat /tmp/data.csv
    args:
      - {inputValue: url}
    output:
      - path: /tmp/data.csv
```

### Step 3: Create Pipeline

**pipeline.py**:
```python
import kfp
from kfp import dsl
from kfp.components import load_component_from_file

@kfp.dsl.pipeline(name='simple-ml-pipeline', description='My first pipeline')
def my_pipeline(url: str, epochs: int = 10):
    download = load_component_from_file('download/component.yaml')(url=url)
    preprocess = load_component_from_file('preprocess/component.yaml')(data=download.output)
    train = load_component_from_file('train/component.yaml')(data=preprocess.output, epochs=epochs)
```

### Step 4: Compile Pipeline

```bash
dsl-compile --py pipeline.py --output pipeline.yaml
```

### Step 5: Upload and Run

1. Open Kubeflow Dashboard
2. Go to Pipelines → Upload pipeline
3. Upload `pipeline.yaml`
4. Create run with parameters

---

## Lesson 2.3: Running and Monitoring Pipelines

### Pipeline Run Lifecycle

```
Pending → Running → Succeeded/Failed
              ↓
         Cancelled
```

### Monitoring in Dashboard

| View | What It Shows |
|------|---------------|
| Graph View | Pipeline structure with status |
| Table View | List of runs with status |
| Run Details | Logs, outputs, duration |

### Viewing Logs

```bash
# Get pod name
kubectl get pods -n kubeflow | grep <run-name>

# View logs
kubectl logs <pod-name> -n kubeflow
```

### Common Issues

| Issue | Solution |
|-------|----------|
| Component stuck | Check image exists, resources sufficient |
| OOM error | Increase memory limits |
| Image pull error | Check registry access |

---

## Lesson 2.4: Pipeline Parameters

### Parameter Types

- **Input parameters**: Values passed into pipeline
- **Output parameters**: Values returned from components
- **Artifacts**: Large data (datasets, models)

### Example with Parameters

```python
@kfp.dsl.pipeline(name='parameterized-pipeline')
def train_pipeline(
    learning_rate: float = 0.01,
    batch_size: int = 32,
    epochs: int = 10
):
    # Components use these parameters
    model = train_op(
        lr=learning_rate,
        batch_size=batch_size,
        epochs=epochs
    )
```

### Passing Parameters Between Components

```python
def preprocess_pipeline(normalize: bool):
    config = create_config_op(normalize=normalize)
    data = load_data_op()
    processed = preprocess_op(data=data.output, config=config.output)
```

---

## Lesson 2.5: Component Reusability

### Why Reuse Components?

1. **Share across pipelines** - Same preprocessing for different models
2. **Team collaboration** - Standard components for everyone
3. **Testing** - Test component once, use many places

### Component Registry

Store components in a registry:
```
components/
├── download/
│   └── component.yaml
├── preprocess/
│   └── component.yaml
└── train/
    └── component.yaml
```

### Loading Components

```python
# From file
load_component_from_file('components/train/component.yaml')

# From URL
load_component_from_file('https://example.com/components/train.yaml')

# From package
kfp.components.import_func('google-cloud-storage')
```

---

## Key Takeaways

1. **Pipeline** = connected components forming a DAG
2. **Component** = single executable task
3. **Run** = single pipeline execution
4. **Parameters** = control pipeline behavior
5. **Artifacts** = pass data between components

---

## Next Section

Section 3: Data & Artifact Management
- Learn about MinIO for storage
- Pass data between pipeline components
- Track metadata and lineage
