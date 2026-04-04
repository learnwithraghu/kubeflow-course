# Exercise 5: Run a Katib Experiment

## Objective
Create and run a hyperparameter tuning experiment using Katib.

## Tasks

### Task 1: Create Training Script

**train.py**:
```python
import os
import numpy as np

lr = float(os.environ.get('LR', '0.01'))
batch_size = int(os.environ.get('BS', '32'))
epochs = int(os.environ.get('EPOCHS', '5'))

print(f"Training with LR={lr}, BS={batch_size}, EPOCHS={epochs}")

# Simulate training
for epoch in range(epochs):
    # Simulate loss (decreases with LR)
    loss = 1.0 / ((epoch + 1) * lr * 10)
    accuracy = 1.0 - (loss * 0.1) + np.random.normal(0, 0.01)
    accuracy = max(0, min(1, accuracy))
    print(f"Epoch {epoch+1}: Loss={loss:.4f}, Accuracy={accuracy:.4f}")

final_accuracy = 0.95 + np.random.normal(0, 0.02)
print(f"Final accuracy: {final_accuracy:.4f}")
```

### Task 2: Create Docker Image

```dockerfile
FROM python:3.9
WORKDIR /opt
COPY train.py /opt/
CMD ["python", "/opt/train.py"]
```

```bash
docker build -t your-registry/hpo-train:latest .
docker push your-registry/hpo-train:latest
```

### Task 3: Create Katib Experiment

**experiment.yaml**:
```yaml
apiVersion: kubeflow.org/v1
kind: Experiment
metadata:
  name: hpo-experiment
  namespace: kubeflow
spec:
  objective:
    type: maximize
    goal: 0.95
    objectiveMetricName: accuracy
  algorithm:
    algorithmName: random
  maxTrialCount: 6
  parallelTrialCount: 2
  parameters:
    - name: LR
      parameterType: double
      feasibleSpace:
        min: "0.001"
        max: "0.1"
    - name: BS
      parameterType: categorical
      feasibleSpace:
        list: ["16", "32", "64"]
    - name: EPOCHS
      parameterType: categorical
      feasibleSpace:
        list: ["3", "5", "10"]
  trialTemplate:
    trialParameters:
      - name: LR
        reference: LR
      - name: BS
        reference: BS
      - name: EPOCHS
        reference: EPOCHS
    trialSpec:
      apiVersion: batch/v1
      kind: Job
      spec:
        template:
          spec:
            containers:
              - name: training
                image: your-registry/hpo-train:latest
                env:
                  - name: LR
                    value: $(TrialParams.LR)
                  - name: BS
                    value: $(TrialParams.BS)
                  - name: EPOCHS
                    value: $(TrialParams.EPOCHS)
```

### Task 4: Run and Monitor

```bash
# Create experiment
kubectl apply -f experiment.yaml

# Check experiment status
kubectl get experiments -n kubeflow

# Watch trials
kubectl get trials -n kubeflow -w

# Describe experiment
kubectl describe experiment hpo-experiment -n kubeflow
```

### Task 5: View Results

Open Kubeflow Dashboard → Katib section to see:
- Trials table with all configurations
- Best hyperparameters found
- Accuracy chart over trials

## Success Criteria

- [ ] Experiment created
- [ ] Multiple trials run in parallel
- [ ] Best configuration identified
- [ ] Results visible in dashboard

## Cleanup

```bash
kubectl delete experiment hpo-experiment -n kubeflow
```

## Challenge

Try changing `algorithmName` to `bayesian` and compare results!
