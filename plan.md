# Kubeflow Student Syllabus

## 7-Section Course Outline (Beginner-Friendly)

### Section 1: Kubeflow Fundamentals
- What is Kubeflow and why use it?
- Kubeflow architecture overview
- Core components introduction
- Local setup with k3s

### Section 2: Kubeflow Pipelines (KFP)
- Pipeline concepts and terminology
- Building your first simple pipeline
- Components vs Pipeline DSL
- Running and monitoring pipelines

### Section 3: Data & Artifact Management
- Kubeflow pipeline artifacts
- MinIO for local storage
- Metadata tracking basics
- Passing data between components

### Section 4: Training with Kubeflow
- Training jobs on Kubernetes
- Distributed training concepts
- PyTorch/TensorFlow operators
- GPU basics (if available)

### Section 5: Hyperparameter Tuning with Katib
- Introduction to HPO
- Katib search spaces
- Running experiments
- Analyzing results

### Section 6: Model Serving with KServe
- What is model serving?
- Deploying inference services
- Making predictions
- Monitoring and scaling

### Section 7: MLOps with Kubeflow
- End-to-end workflow automation
- Experiment tracking
- Pipeline triggers
- Best practices summary

---

## Practical Exercises (Per Section)

| Section | Exercise | Complexity |
|---------|----------|------------|
| 1 | Install Kubeflow on k3s | Basic |
| 2 | Build 3-step ML pipeline | Basic |
| 3 | Pass data between components | Basic |
| 4 | Run single-node training job | Intermediate |
| 5 | Tune learning rate with Katib | Intermediate |
| 6 | Deploy model for inference | Intermediate |
| 7 | Connect full pipeline | Advanced |

---

## Next Steps
1. Create detailed lesson plans for each section
2. Create demo repository structure
3. Prepare prerequisite scripts for students

---

## Proposed 6 Demos

### Demo 1: End-to-End ML Pipeline (MNIST Classification)
**Purpose**: Showcase the core of Kubeflow - Kubeflow Pipelines (KFP)
**Components**: Data ingestion → Preprocessing → Training → Evaluation → Model registration
**Key Features**: Reusable components, pipeline versioning, run history
**Duration**: ~20-30 min

### Demo 2: Automated Hyperparameter Tuning (Katib)
**Purpose**: Demonstrate automatic HPO without manual trial-and-error
**Components**: Same MNIST model, define search space (learning rate, layers, optimizer)
**Key Features**: Bayesian optimization, early stopping, parallel trials
**Duration**: ~15-20 min

### Demo 3: Distributed Training (PyTorch/TensorFlow)
**Purpose**: Show scalable training across multiple worker nodes
**Components**: Distributed data parallel training on Kubernetes
**Key Features**: GPU scheduling, worker coordination, fault tolerance
**Duration**: ~20-25 min

### Demo 4: Model Serving & Inference (KServe)
**Purpose**: Deploy trained model as REST/gRPC API endpoint
**Components**: Train model → Save to model registry → Deploy with KServe → Inference
**Key Features**: Auto-scaling, canary deployments, model explainability
**Duration**: ~15-20 min

### Demo 5: Data & Feature Management
**Purpose**: Show ML asset versioning and reproducibility
**Components**: Dataset versioning, feature transformations, lineage tracking
**Key Features**: KFP artifacts, metadata tracking, experiment comparison
**Duration**: ~15 min

### Demo 6: Pipeline Composition & Automation
**Purpose**: Orchestrate multi-stage workflows with triggers
**Components**: Trigger-based pipeline execution, conditional steps, notifications
**Key Features**: Cron triggers, webhook integration, Slack notifications
**Duration**: ~15-20 min

---

## Implementation Order (Self-Contained Demos)

| Demo | Focus Area | Complexity | Setup Priority |
|------|------------|------------|----------------|
| 1 | KFP Basics - Build first pipeline | Beginner | HIGH |
| 2 | Katib - Hyperparameter tuning | Beginner | MEDIUM |
| 3 | Distributed Training (CPU simulation) | Intermediate | MEDIUM |
| 4 | KServe - Model serving | Intermediate | HIGH |
| 5 | ML Metadata - Experiment tracking | Intermediate | LOW |
| 6 | Pipeline Automation - Triggers | Advanced | LOW |

---

## Prerequisites (Per Demo)

### Demo 1-2 Prerequisites
- Kubeflow Pipelines (KFP) installed
- MinIO or equivalent for artifact storage
- Sample datasets (MNIST, CIFAR-10)

### Demo 3 Prerequisites
- Kubernetes cluster with multiple nodes OR
- Single node with sufficient CPU cores for worker simulation
- PyTorch/TensorFlow distributed builds

### Demo 4 Prerequisites
- KServe installed
- InferenceService CRD
- Trained model from Demo 1

### Demo 5-6 Prerequisites
- Kubeflow Metadata SDK
- KFP hooks/triggers

---

## Local Setup Requirements

### Must Have
- [ ] Kubernetes cluster (k3s, minikube, or kind)
- [ ] Docker container runtime
- [ ] kubectl configured
- [ ] 8GB+ RAM
- [ ] 4+ CPU cores

### Recommended
- [ ] NVIDIA GPU (optional for Demo 3)
- [ ] 50GB+ disk space

---

## File Structure for Demos

```
kubeflow-demos/
├── demo1-mnist-pipeline/
│   ├── components/          # Reusable KFP components
│   ├── pipelines/           # Pipeline definitions
│   └── notebooks/           # E2E Jupyter notebook
├── demo2-katib-hpo/
│   ├── experiments/          # HPO configurations
│   └── searchSpaces/         # Hyperparameter spaces
├── demo3-distributed-training/
│   ├── pytorch-job/          # PyTorch distributed
│   └── tf-job/              # TensorFlow distributed
├── demo4-kserve-inference/
│   ├── models/               # Saved models
│   └── InferenceService/     # KServe configs
├── demo5-ml-metadata/
│   └── experiments/          # Tracking configs
└── demo6-pipeline-automation/
    └── triggers/             # Trigger definitions
```

---

## Questions/Clarifications Needed

1. ~~Kubernetes Setup~~: k3s selected
2. ~~Kubeflow Version~~: Latest stable
3. ~~ML Framework~~: PyTorch (simple and works well)
4. ~~GPU Resources~~: CPU only - will simulate distributed training
5. ~~Demo Depth~~: Self-contained demos (each independent)
6. **Student Level**: _[Please confirm: Beginner, intermediate, or advanced?]_

---

## Updated Plan Based on Responses

### Environment
- **Kubernetes**: k3s (lightweight, easy setup)
- **ML Framework**: PyTorch (simple, widely used)
- **Training**: CPU-only (distributed training simulated)
- **Demo Style**: Self-contained (each demo independent)

### Focus Shifts
- Demo 3 (Distributed Training): Will use CPU workers simulation, no GPU required
- All demos: Keep simple, focus on core concepts over complexity

---

## Next Steps (After Plan Approval)

1. Create demo1-mnist-pipeline with reusable KFP components
2. Set up MinIO for artifact storage
3. Create sample MNIST dataset preprocessing components
4. Build and verify first pipeline end-to-end
5. Document each demo with README and expected outputs
