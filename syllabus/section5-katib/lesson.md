# Section 5: Hyperparameter Tuning with Katib

## Learning Objectives
- Understand hyperparameter optimization (HPO)
- Define search spaces for Katib
- Run and analyze experiments
- Compare different configurations

---

## Lesson 5.1: Introduction to HPO

### What is Hyperparameter Optimization?

HPO is the process of **automatically finding the best hyperparameters** for an ML model.

### Why HPO?

| Manual Tuning | Automated HPO |
|--------------|--------------|
| Time-consuming | Faster |
| May miss optimal | Explores systematically |
| Not reproducible | Reproducible |
| Expert needed | Requires minimal domain knowledge |

### Katib Overview

Katib is Kubeflow's HPO system based on **Google Vizier**.

### Supported Algorithms

| Algorithm | Best For |
|-----------|----------|
| **Random** | Quick baseline |
| **Grid** | Small search spaces |
| **Bayesian** | Efficient optimization |
| **Hyperband** | Early stopping |
| **NAS** | Architecture search |

---

## Lesson 5.2: Katib Concepts

### Key Terms

| Term | Definition |
|------|------------|
| **Experiment** | Single HPO run with goal |
| **Trial** | One training run with specific params |
| **Search Space** | Range of values to search |
| **Objective** | Metric to optimize |
| **Suggestion** | Proposed hyperparameter config |

### Experiment CRD

```yaml
apiVersion: kubeflow.org/v1
kind: Experiment
metadata:
  name: hpo-experiment
  namespace: kubeflow
spec:
  objective:
    type: maximize
    goal: 0.99
    objectiveMetricName: accuracy
  algorithm:
    algorithmName: bayesian
  parameters:
    - name: learning_rate
      parameterType: double
      feasibleSpace:
        min: "0.001"
        max: "0.1"
    - name: batch_size
      parameterType: categorical
      feasibleSpace:
        list: ["16", "32", "64"]
  trialTemplate:
    trialParameters:
      - name: lr
        description: Learning rate
        reference: learning_rate
    trialSpec:
      apiVersion: batch/v1
      kind: Job
      spec:
        template:
          spec:
            containers:
              - name: training
                image: pytorch/pytorch:latest
                command:
                  - python
                  - /opt/train.py
                  - --lr=$(TrialParams.lr)
```

---

## Lesson 5.3: Creating Experiments

### Define Search Space

```yaml
parameters:
  - name: learning_rate
    parameterType: double
    feasibleSpace:
      min: "0.0001"
      max: "0.1"
  - name: num_layers
    parameterType: int
    feasibleSpace:
      min: "1"
      max: "5"
  - name: optimizer
    parameterType: categorical
    feasibleSpace:
      list: ["adam", "sgd", "rmsprop"]
```

### Define Objective

```yaml
objective:
  type: maximize  # or minimize
  goal: 0.95
  objectiveMetricName: accuracy
  additionalMetricNames:
    - loss
```

### Complete Experiment YAML

```yaml
apiVersion: kubeflow.org/v1
kind: Experiment
metadata:
  name: mnist-hpo
  namespace: kubeflow
spec:
  objective:
    type: maximize
    goal: 0.98
    objectiveMetricName: accuracy
  algorithm:
    algorithmName: bayesian
  maxTrialCount: 10
  parallelTrialCount: 3
  parameters:
    - name: learning_rate
      parameterType: double
      feasibleSpace:
        min: "0.001"
        max: "0.1"
    - name: batch_size
      parameterType: categorical
      feasibleSpace:
        list: ["16", "32", "64"]
  trialTemplate:
    trialParameters:
      - name: lr
        reference: learning_rate
      - name: bs
        reference: batch_size
    trialSpec:
      apiVersion: batch/v1
      kind: Job
      spec:
        template:
          spec:
            containers:
              - name: training
                image: your-training-image
                env:
                  - name: LR
                    value: $(TrialParams.lr)
                  - name: BS
                    value: $(TrialParams.bs)
```

---

## Lesson 5.4: Running and Analyzing Experiments

### Create Experiment

```bash
kubectl apply -f experiment.yaml
```

### Check Status

```bash
# List experiments
kubectl get experiments -n kubeflow

# Get experiment details
kubectl describe experiment mnist-hpo -n kubeflow
```

### View Trials

```bash
# List all trials
kubectl get trials -n kubeflow

# View specific trial
kubectl describe trial mnist-hpo-xxxxx -n kubeflow
```

### Dashboard View

1. Go to **Katib** section in Kubeflow dashboard
2. Select your experiment
3. View:
   - **Trials table**: All runs with metrics
   - **Best hyperparameters**: Current best config
   - **Optimization history**: Chart of results

### Analyze Results

```bash
# Get best trial
kubectl get trials -n kubeflow -o jsonpath='{.items[?(@.status.conditions[0].type=="Succeeded")]}' | jq '.[] | select(.status.observationMetrics.accuracy == "0.98")'
```

---

## Lesson 5.5: Early Stopping

### Early Stopping with Hyperband

```yaml
spec:
  algorithm:
    algorithmName: hyperband
  earlyStopping:
    algorithmName: median
    earlyStoppingParameters:
      minTrialsRequired: 3
      startLoss: 0.9
```

### Median Stopping Rule

Stops a trial if its performance is worse than the median of all running trials.

---

## Key Takeaways

1. **Katib** = Kubeflow's HPO system
2. **Experiment** = defines search space and objective
3. **Trial** = single training run
4. **Algorithms** = Random, Grid, Bayesian, Hyperband
5. **Dashboard** = view results and best config

---

## Next Section

Section 6: Model Serving with KServe
- Deploy trained models as REST APIs
- Configure InferenceService
- Make predictions and monitor
