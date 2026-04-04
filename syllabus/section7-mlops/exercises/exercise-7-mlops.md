# Exercise 7: End-to-End Pipeline with Automation

## Objective
Create a pipeline that trains and deploys a model automatically.

## Tasks

### Task 1: Create Complete Pipeline

**pipeline.py**:
```python
import kfp
from kfp import dsl
from kfp.components import load_component_from_file

@dsl.pipeline(name='auto-train-serve')
def auto_pipeline(
    data_url: str,
    epochs: int = 5,
    deploy_threshold: float = 0.9
):
    # Download data
    data = load_component_from_file('components/download/component.yaml')(url=data_url)
    
    # Preprocess
    processed = load_component_from_file('components/preprocess/component.yaml')(data=data.output)
    
    # Train
    model = load_component_from_file('components/train/component.yaml')(
        data=processed.output,
        epochs=epochs
    )
    
    # Evaluate
    metrics = load_component_from_file('components/evaluate/component.yaml')(
        model=model.output,
        data=processed.output
    )
    
    # Deploy conditionally
    with dsl.Condition(metrics.outputs['accuracy'] > deploy_threshold):
        load_component_from_file('components/deploy/component.yaml')(model=model.output)
```

### Task 2: Create Evaluation Component

**components/evaluate/component.yaml**:
```yaml
name: Evaluate Model
description: Evaluate model and output metrics
inputs:
  - {name: model, type: Model}
  - {name: data, type: Dataset}
outputs:
  - {name: metrics, type: Metrics}
implementation:
  container:
    image: python:3.9
    command:
      - sh
      - -c
      - |
        python3 -c "
        import json
        # Simulate evaluation
        metrics = {'accuracy': 0.95, 'f1': 0.93}
        with open('/tmp/metrics.json', 'w') as f:
            json.dump(metrics, f)
        print('Metrics:', metrics)
        "
    args:
      - {inputPath: model}
      - {inputPath: data}
      - {outputPath: metrics}
```

### Task 3: Create Deployment Component

**components/deploy/component.yaml**:
```yaml
name: Deploy Model
description: Deploy model to KServe
inputs:
  - {name: model, type: Model}
implementation:
  container:
    image: python:3.9
    command:
      - sh
      - -c
      - |
        echo "Deploying model..."
        python3 -c "
        import yaml
        isvc = {
            'apiVersion': 'serving.kserve.io/v1',
            'kind': 'InferenceService',
            'metadata': {'name': 'auto-classifier', 'namespace': 'kubeflow'},
            'spec': {
                'predictor': {
                    'pytorch': {
                        'storageUri': 's3://models/classifier'
                    }
                }
            }
        }
        print('Would deploy:', isvc['metadata']['name'])
        # In real scenario, apply via kubectl
        "
    args:
      - {inputPath: model}
```

### Task 4: Compile and Upload

```bash
kfp compiler pipeline.py --output auto-pipeline.yaml
```

Upload to Kubeflow dashboard.

### Task 5: Create Recurring Schedule

In Kubeflow Dashboard:
1. Go to Pipelines → Your pipeline
2. Click "Create run" → "Schedule"
3. Set recurring options:
   - **Type**: Periodic
   - **Interval**: Daily
   - **Run name**: auto-train-daily

### Task 6: Verify Automation

```bash
# Check scheduled workflow
kubectl get scheduledworkflows -n kubeflow

# View recent runs
kubectl get runs -n kubeflow
```

## Success Criteria

- [ ] Pipeline compiles successfully
- [ ] Conditional deployment works
- [ ] Schedule created
- [ ] Pipeline runs on schedule

## Cleanup

```bash
kubectl delete -f auto-pipeline.yaml -n kubeflow
```

## Challenge

Add Slack notification on pipeline completion!
