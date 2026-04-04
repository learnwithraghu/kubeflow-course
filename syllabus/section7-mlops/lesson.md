# Section 7: MLOps & Automation

## Learning Objectives
- Connect end-to-end ML workflows
- Automate pipeline execution with triggers
- Track experiments and compare runs
- Apply MLOps best practices

---

## Lesson 7.1: End-to-End Workflow

### Complete ML Lifecycle

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│   Data   │───▶│  Train   │───▶│ Evaluate │───▶│  Serve   │
│  Ingest  │    │  Model  │    │  Model   │    │  Model   │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
     │               │               │               │
     └───────────────┴───────────────┴───────────────┘
                      │
                 ┌──────────┐
                 │ Monitor  │
                 │ & Alert  │
                 └──────────┘
```

### Connecting Pipeline to Serving

```python
@kfp.dsl.pipeline(name='e2e-ml')
def e2e_pipeline():
    # Data processing
    data = download_op(url=data_url)
    processed = preprocess_op(data=data.output)
    
    # Training
    model = train_op(data=processed.output, epochs=epochs)
    
    # Evaluation
    metrics = evaluate_op(model=model.output, data=processed.output)
    
    # Conditional deployment
    with kfp.dsl.Condition(metrics.outputs['accuracy'] > 0.95):
        deploy_op(model=model.output)
```

---

## Lesson 7.2: Experiment Tracking

### Why Track Experiments?

| Without Tracking | With Tracking |
|-----------------|---------------|
| Lost runs | All runs logged |
| Can't compare | Side-by-side comparison |
| No reproducibility | Reproducible experiments |
| Manual notes | Automatic metadata |

### KFP Experiment Management

```bash
# Create experiment
client = kfp.Client()
client.create_experiment('mnist-experiments')

# List experiments
client.list_experiments()

# Run within experiment
client.run_pipeline(
    experiment_name='mnist-experiments',
    job_name='run-001',
    pipeline_id='pipeline-id'
)
```

### Compare Runs

In Kubeflow Dashboard:
1. Go to Experiments
2. Select multiple runs
3. Click Compare
4. View metrics side-by-side

### Log Metrics in Component

```python
def train_component(output_dir: str, epochs: int):
    # Training code...
    accuracy = train_model()
    
    # Log metrics
    from kubeflow.metadata import metadata
    metadata.Store(
        artifact_type="run_metrics",
        name=f"run-{output_dir}",
        uri=output_dir,
        properties={
            "accuracy": float(accuracy),
            "epochs": epochs
        }
    )
```

---

## Lesson 7.3: Pipeline Automation

### Recurring Runs (Cron)

```yaml
apiVersion: kubeflow.org/v1beta1
kind: ScheduledWorkflow
metadata:
  name: daily-training
spec:
  schedule: "0 2 * * *"  # 2 AM daily
  pipelineSpec:
    pipelineId: pipeline-id
    parameters:
      epochs: "10"
```

### Event-Driven Triggers

```yaml
apiVersion: eventing.knative.dev/v1
kind: Trigger
metadata:
  name: data-trigger
spec:
  broker: default
  filter:
    attributes:
      type: google.cloud.storage.object.v1.finalized
  subscriber:
    ref:
      apiVersion: serving.kserve.io/v1
      kind: Action
      name: training-trigger
```

### Webhook Integration

```python
from flask import Flask, request

app = Flask(__name__)

@app.route('/trigger', methods=['POST'])
def trigger_pipeline():
    data = request.json
    
    # Trigger KFP run
    client.run_pipeline(
        pipeline_id='pipeline-id',
        parameters=data
    )
    
    return {"status": "triggered"}
```

### Notification Integration

```yaml
apiVersion: kubeflow.org/v1beta1
kind: ScheduledWorkflow
metadata:
  name: nightly-training
spec:
  workflowSpec:
    templates:
      - name: pipeline-template
        # ... pipeline steps ...
        outputs:
          artifacts:
            - name: metrics
              path: /metrics.json
  notifications:
    onSuccess:
      - webhook:
          url: https://hooks.slack.com/services/xxx
          body: |
            {"text": "Training completed successfully!"}
    onFailure:
      - webhook:
          url: https://hooks.slack.com/services/xxx
          body: |
            {"text": "Training failed!"}
```

---

## Lesson 7.4: CI/CD for ML

### GitOps for ML

```yaml
# .github/workflows/ml-pipeline.yml
name: ML Pipeline

on:
  push:
    branches: [main]
    paths: ['data/**', 'src/**']

jobs:
  train:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run Kubeflow Pipeline
        run: |
          kfp run submit \
            --experiment-name=github-actions \
            --pipeline-id=${{ secrets.PIPELINE_ID }} \
            --params=epochs=10
```

### Model Registry

```bash
# Register new model
model-registry register \
  --name=classifier-v2 \
  --uri=s3://models/classifier-v2 \
  --model-format=pytorch \
  --version=2

# List registered models
model-registry list
```

---

## Lesson 7.5: MLOps Best Practices

### Reproducibility

| Practice | Implementation |
|---------|---------------|
| Code versioning | Git for all code |
| Data versioning | DVC or similar |
| Environment | Container images |
| Config management | YAML/JSON configs |

### Configuration Example

```yaml
# config.yaml
experiment:
  name: mnist-classifier
  seed: 42

data:
  url: s3://data/mnist
  version: v1

training:
  epochs: 10
  batch_size: 32
  learning_rate: 0.001
  optimizer: adam

model:
  layers: [784, 128, 64, 10]
  activation: relu
```

### Documentation

```markdown
# Model Card: MNIST Classifier v1

## Overview
- **Purpose**: Handwritten digit classification
- **Framework**: PyTorch 1.9
- **Created**: 2024-01-15

## Training
- **Dataset**: MNIST (60k train, 10k test)
- **Epochs**: 10
- **Final Accuracy**: 98.5%

## Limitations
- Poor on rotated digits
- Assumes clean input images
```

---

## Key Takeaways

1. **E2E Workflow** = Connect data → train → evaluate → serve
2. **Experiment Tracking** = Log all runs, compare easily
3. **Automation** = Cron triggers, webhooks, notifications
4. **CI/CD** = GitOps for ML with automated training
5. **Best Practices** = Reproducibility, versioning, documentation

---

## Course Summary

Congratulations! You've completed the Kubeflow syllabus.

### What You Learned

| Section | Topic | Skills |
|---------|-------|--------|
| 1 | Fundamentals | Install & navigate Kubeflow |
| 2 | KFP | Build ML pipelines |
| 3 | Data & Artifacts | Manage data flow |
| 4 | Training | Run training jobs |
| 5 | Katib | Hyperparameter tuning |
| 6 | KServe | Model serving |
| 7 | MLOps | Automation & best practices |

### Next Steps

1. **Practice**: Build your own pipelines
2. **Explore**: Try different frameworks (TensorFlow, XGBoost)
3. **Scale**: Deploy to cloud (AWS, GCP, Azure)
4. **Contribute**: Join Kubeflow community

### Resources

- [Kubeflow Documentation](https://www.kubeflow.org/docs/)
- [Kubeflow Slack](https://join.slack.com/t/kubeflow/shared_invite)
- [Kubeflow GitHub](https://github.com/kubeflow/kubeflow)
