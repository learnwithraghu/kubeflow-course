# Kubeflow Student Syllabus

A beginner-friendly 7-section course for learning Kubeflow on local Kubernetes.

**Prerequisites**: Basic Python, Docker, and Kubernetes knowledge

---

## Course Overview

| Section | Topic | Time | Difficulty |
|---------|-------|------|------------|
| 1 | Kubeflow Fundamentals | 45 min | Beginner |
| 2 | Kubeflow Pipelines (KFP) | 60 min | Beginner |
| 3 | Data & Artifact Management | 45 min | Beginner |
| 4 | Training with Kubeflow | 60 min | Intermediate |
| 5 | Hyperparameter Tuning (Katib) | 45 min | Intermediate |
| 6 | Model Serving (KServe) | 45 min | Intermediate |
| 7 | MLOps & Automation | 45 min | Advanced |

**Total**: ~5.5 hours of hands-on content

---

## Section 1: Kubeflow Fundamentals

### Learning Objectives
- Understand what Kubeflow is and its purpose
- Know the core components of Kubeflow
- Set up Kubeflow on local k3s cluster

### Topics
1. **What is Kubeflow?**
   - MLOps platform for Kubernetes
   - Why Kubeflow? (portability, scalability, experimentation)
   - Kubeflow vs other platforms (MLflow, Airflow)

2. **Kubeflow Architecture**
   - Control plane vs data plane
   - Namespace isolation
   - Authentication and RBAC

3. **Core Components**
   - Kubeflow Pipelines (KFP)
   - Katib (hyperparameter tuning)
   - KServe (model serving)
   - Central Dashboard
   - JupyterHub (notebook servers)

4. **Local Setup**
   - Installing k3s
   - Installing Kubeflow with k3d
   - Accessing Kubeflow dashboard

### Exercises
- [ ] Install k3s on local machine
- [ ] Deploy Kubeflow using k3d
- [ ] Access and explore Kubeflow dashboard
- [ ] Create a namespace for your project

### Resources
- [Kubeflow Documentation](https://www.kubeflow.org/docs/)
- [k3d Documentation](https://k3d.io/)

---

## Section 2: Kubeflow Pipelines (KFP)

### Learning Objectives
- Understand pipeline concepts and terminology
- Build a simple 3-step ML pipeline
- Run pipeline and monitor execution

### Topics
1. **Pipeline Concepts**
   - Components (containerized ML tasks)
   - Pipeline DSL (domain-specific language)
   - DAG (directed acyclic graph)
   - Pipeline parameters and artifacts

2. **Building Blocks**
   - Component definition (YAML + container)
   - Pipeline definition (Python SDK)
   - Artifact passing between components

3. **Running Pipelines**
   - Compile and upload
   - Create runs
   - Monitor execution
   - View logs and outputs

4. **Hello World Pipeline**
   ```python
   # Simple 3-step pipeline: download → transform → train
   download_data() → transform_data() → train_model()
   ```

### Exercises
- [ ] Create a simple "Hello World" component
- [ ] Build a 3-step data processing pipeline
- [ ] Run pipeline and view results
- [ ] Modify pipeline parameters

### Resources
- [KFP SDK Documentation](https://www.kubeflow.org/docs/components/pipelines/sdk/)
- [KFP SDK GitHub](https://github.com/kubeflow/pipelines)

---

## Section 3: Data & Artifact Management

### Learning Objectives
- Understand Kubeflow's artifact system
- Set up MinIO for local storage
- Pass data between pipeline components

### Topics
1. **Artifacts in Kubeflow**
   - What are artifacts?
   - Artifact types (datasets, models, metrics)
   - Metadata and lineage tracking

2. **MinIO Setup**
   - Object storage basics
   - Configuring MinIO for Kubeflow
   - Bucket management

3. **Data Flow in Pipelines**
   - Input/output definitions
   - Artifact passing
   - Volume mounts for large data

4. **Metadata Tracking**
   - Kubeflow Metadata SDK
   - Recording experiments
   - Querying metadata

### Exercises
- [ ] Set up MinIO on Kubernetes
- [ ] Configure Kubeflow to use MinIO
- [ ] Create pipeline that passes data between components
- [ ] View artifact lineage in dashboard

### Resources
- [MinIO Documentation](https://min.io/docs/minio/kubernetes/upstream/)
- [Kubeflow Metadata](https://www.kubeflow.org/docs/components/metadata/)

---

## Section 4: Training with Kubeflow

### Learning Objectives
- Run ML training jobs on Kubernetes
- Understand distributed training concepts
- Use PyTorch/TensorFlow operators

### Topics
1. **Training Jobs**
   - TFJob (TensorFlow)
   - PyTorchJob (PyTorch)
   - MPIJob (distributed)
   - Job concepts: master, worker, chief

2. **Single-Node Training**
   - Define training job
   - Configure resources (CPU/memory)
   - Monitor training progress

3. **Distributed Training**
   - Data parallel training
   - Model parallel training
   - Worker coordination

4. **GPU Support** (if available)
   - GPU device plugin
   - Scheduling GPU workloads
   - Multi-GPU training

### Exercises
- [ ] Create a single-node PyTorch training job
- [ ] Run distributed training with 2 workers
- [ ] Monitor training with TensorBoard
- [ ] Save model checkpoint to MinIO

### Resources
- [Kubeflow Training Operators](https://www.kubeflow.org/docs/components/training/)
- [PyTorchJob Documentation](https://www.kubeflow.org/docs/components/training/pytorch/)

---

## Section 5: Hyperparameter Tuning with Katib

### Learning Objectives
- Understand hyperparameter optimization (HPO)
- Define search spaces for Katib
- Run and analyze experiments

### Topics
1. **Introduction to HPO**
   - Why HPO matters
   - Manual vs automated tuning
   - Katib overview

2. **Katib Concepts**
   - Experiment CRD
   - Search spaces (discrete, continuous)
   - Search algorithms (random, grid, Bayesian)
   - Trial scheduler

3. **Creating Experiments**
   - Objective metrics
   - Parameter ranges
   - Trial template
   - Success criteria

4. **Analyzing Results**
   - Best hyperparameters
   - Training curves
   - Comparison with baseline

### Exercises
- [ ] Define search space for MNIST training
- [ ] Create Katib experiment
- [ ] Run experiment with 10 trials
- [ ] Analyze results and find best params

### Resources
- [Katib Documentation](https://www.kubeflow.org/docs/components/katib/)
- [Katib GitHub](https://github.com/kubeflow/katib)

---

## Section 6: Model Serving with KServe

### Learning Objectives
- Deploy trained models as REST APIs
- Understand inference service configuration
- Make predictions and monitor performance

### Topics
1. **Model Serving Concepts**
   - What is KServe?
   - InferenceService CRD
   - Predictor types (torchserve, triton, sklearn)

2. **Deployment**
   - Creating InferenceService
   - Model storage (MinIO)
   - Resource configuration
   - Version management

3. **Inference**
   - REST API usage
   - Request/response formats
   - Batch prediction
   - Preprocessing transforms

4. **Monitoring & Scaling**
   - Auto-scaling (Knative)
   - Monitoring with Prometheus
   - Canary deployments
   - A/B testing

### Exercises
- [ ] Deploy model from Section 4 as InferenceService
- [ ] Test prediction endpoint
- [ ] Configure auto-scaling
- [ ] Set up canary deployment

### Resources
- [KServe Documentation](https://kserve.github.io/website/)
- [KServe GitHub](https://github.com/kserve/kserve)

---

## Section 7: MLOps & Automation

### Learning Objectives
- Connect end-to-end ML workflows
- Automate pipeline execution
- Apply MLOps best practices

### Topics
1. **End-to-End Workflow**
   - Connecting pipelines to serving
   - Automated retraining
   - Model registry integration

2. **Pipeline Automation**
   - Recurring runs (cron)
   - Event-driven triggers
   - Webhook integrations
   - Notifications

3. **Experiment Tracking**
   - Logging metrics
   - Comparing runs
   - Reproducibility
   - Collaboration

4. **MLOps Best Practices**
   - Code reproducibility
   - Data versioning
   - Model versioning
   - CI/CD for ML

### Exercises
- [ ] Create pipeline that trains and deploys automatically
- [ ] Set up cron trigger for nightly training
- [ ] Integrate Slack notifications
- [ ] Document your MLOps workflow

### Resources
- [Kubeflow Pipelines SDK](https://www.kubeflow.org/docs/components/pipelines/sdk/)
- [MLOps Principles](https://mlops.org/)

---

## Quick Reference

### Common Commands

```bash
# List pipelines
kubectl get pipelines -n kubeflow

# List runs
kubectl get runs -n kubeflow

# Check Katib experiments
kubectl get experiments -n kubeflow

# List InferenceServices
kubectl get isvc -n kubeflow

# Check pod status
kubectl get pods -n kubeflow
```

### Port Forwarding

```bash
# Kubeflow Dashboard
kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80

# MinIO Console
kubectl port-forward svc/minio-console -n kubeflow 9001:9090
```

---

## Setup Checklist

Before starting Section 1, ensure you have:

- [ ] macOS/Linux machine
- [ ] Docker installed and running
- [ ] kubectl installed
- [ ] 8GB+ RAM available
- [ ] 20GB+ disk space
- [ ] Internet connection for downloading images

---

*Last updated: 2026-04-04*
