# Kubeflow Student Syllabus

A hands-on course for mastering Kubeflow and MLOps on local Kubernetes. Learn production ML workflows using a real-world use case.

**Real-World Use Case**: E-Commerce Customer Churn Prediction
- Predict which customers are likely to leave
- Build, optimize, and deploy models at scale
- Automate the complete ML lifecycle

**Prerequisites**:
- Basic Python programming (NumPy, Pandas, scikit-learn)
- Docker fundamentals (images, containers, registries)
- Kubernetes basics: pods, deployments, services, namespaces, kubectl commands
- Familiarity with YAML syntax
- Linux/macOS terminal experience

**Recommended**: Review Kubernetes concepts before starting (suggest ~1 hour prep if needed)

---

## Course Overview

| Section | Topic | Hands-On Time | Difficulty | Focus |
|---------|-------|---|------------|-------|
| 1 | Kubeflow & Data Fundamentals | 90 min | Beginner | Setup, architecture, artifact storage |
| 2 | Kubeflow Pipelines (KFP) | 75 min | Beginner | Build reusable ML pipeline components |
| 3 | Advanced Pipeline Design | 60 min | Beginner+ | Complex DAGs, dependencies, parameterization |
| 4 | Training at Scale | 90 min | Intermediate | Single & distributed training jobs |
| 5 | Hyperparameter Optimization (Katib) | 75 min | Intermediate | Automated experimentation & search |
| 6 | Model Serving Essentials | 60 min | Intermediate | KServe basics, inference APIs |
| 7 | MLOps End-to-End | 90 min | Advanced | Automation, monitoring, retraining |

**Total**: ~8.5 hours of hands-on content + 1 hour setup

**Setup Time**: ~45 minutes (k3s/k3d, Kubeflow installation)

---

## Section 1: Kubeflow & Data Fundamentals

### Learning Objectives
- Understand Kubeflow architecture and design patterns
- Deploy Kubeflow on local Kubernetes cluster
- Set up artifact storage (MinIO) for data management
- Explore the course dataset and Kubeflow UI

### Kubeflow Component Architecture

```
┌─────────────────────────────────────────────┐
│         Kubeflow Control Plane              │
├─────────────────────────────────────────────┤
│  • Central Dashboard                        │
│  • KFP (Pipelines) Server                  │
│  • Katib Controller                        │
│  • KServe Controller                       │
│  • Authentication & RBAC                   │
└─────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────┐
│      Data & Artifact Layer (MinIO)          │
│   • Object Storage (S3-compatible)          │
│   • Pipeline artifact tracking              │
│   • Model versioning                        │
└─────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────┐
│      Kubernetes Data Plane                  │
│  • Pod execution                            │
│  • Distributed training (TFJob, PyTorchJob)│
│  • Inference servers (KServe)               │
│  • Resource management                      │
└─────────────────────────────────────────────┘
```

### Topics

1. **What is Kubeflow & Why Kubernetes?**
   - Kubeflow as MLOps orchestration platform
   - Why Kubernetes: reproducibility, scalability, multi-tenancy
   - Kubeflow vs MLflow (single machine) vs Airflow (general orchestration)
   - Real-world production scenarios

2. **Kubeflow Architecture Deep Dive**
   - Control plane: API servers, controllers, dashboards
   - Data plane: Pod execution, storage, networking
   - Namespace isolation & multi-tenancy
   - CRDs (Custom Resource Definitions) for ML objects

3. **Core Components Overview**
   - **KFP (Kubeflow Pipelines)**: Define, compile, and execute ML workflows as DAGs
   - **Katib**: Automated hyperparameter tuning and neural architecture search
   - **Training Operators**: TFJob, PyTorchJob for distributed training
   - **KServe**: Model inference on Kubernetes
   - **Metadata Service**: Experiment tracking and lineage
   - **JupyterHub**: Multi-user notebook access

4. **Artifact Storage with MinIO**
   - Object storage for ML artifacts (datasets, models, metrics)
   - S3-compatible API for seamless integration
   - Bucket organization best practices
   - Metadata tracking via Kubeflow Metadata Service

5. **E-Commerce Customer Churn Dataset**
   - Dataset overview: 10,000 customer records with 12 features
   - Features: account_age, purchase_frequency, avg_order_value, support_tickets, days_since_login, etc.
   - Target: churn (binary: 0=retained, 1=churned)
   - Real-world characteristics: class imbalance, missing values, feature scaling needed
   - Data splits: 70% train, 15% validation, 15% test

### Hands-On Exercises

**Exercise 1.1: Set up Kubeflow Cluster**
- Install k3s/k3d on local machine
- Deploy Kubeflow using k3d manifests (v1.8+)
- Verify all core components are running: `kubectl get pods -n kubeflow`
- Access Kubeflow Dashboard via port-forward
- ✓ Success: Dashboard loads, can see "Pipelines" and "Experiments" sections

**Exercise 1.2: Configure MinIO for Artifact Storage**
- Deploy MinIO in kubeflow namespace
- Create bucket `kubeflow-artifacts`
- Create IAM credentials and ConfigMap for Kubeflow integration
- Verify bucket access via MinIO console
- ✓ Success: MinIO console accessible, bucket created, credentials verified

**Exercise 1.3: Explore Kubeflow Dashboard**
- Create a new namespace: `my-ml-project`
- Navigate through Pipelines, Experiments, Runs sections
- Create a user notebook server via JupyterHub (optional)
- ✓ Success: Namespace created, dashboard responsive, UI elements accessible

**Exercise 1.4: Load Course Dataset**
- Download customer churn CSV to local machine
- Upload dataset to MinIO `kubeflow-artifacts/datasets/` bucket
- Verify file upload and accessibility
- Understand data schema (columns, dtypes, statistics)
- ✓ Success: File accessible via MinIO, basic statistics displayed (mean, std, null counts)

### Debugging & Troubleshooting

| Issue | Diagnosis | Solution |
|-------|-----------|----------|
| k3d cluster creation fails | Check Docker memory allocation | Ensure Docker has 8GB+ RAM, increase in Docker Desktop settings |
| Kubeflow pods stuck in `Pending` | Check node resources | `kubectl describe node` - may need to increase k3d memory via `k3d cluster create --servers-memory 8G` |
| MinIO console unreachable | Port-forward not set | Run `kubectl port-forward svc/minio-console -n kubeflow 9001:9090` |
| Dashboard loading blank | Istio ingress issue | Check Istio: `kubectl get pods -n istio-system`, restart gateway if needed |

### Resources
- [Kubeflow Documentation](https://www.kubeflow.org/docs/)
- [k3d Documentation](https://k3d.io/)
- [MinIO Kubernetes Guide](https://min.io/docs/minio/kubernetes/upstream/)
- [Kubernetes Concepts](https://kubernetes.io/docs/concepts/)

---

## Section 2: Building ML Pipelines with KFP

### Learning Objectives
- Master KFP component creation and containerization
- Build a complete data processing pipeline
- Understand artifact passing and data flow
- Deploy and monitor pipeline executions
- Learn to version and track pipeline artifacts

### Core Concepts

**KFP Components**: Containerized, reusable pipeline steps
```
┌──────────────┐
│  Input Data  │
└──────┬───────┘
       │ (artifact)
       ↓
┌──────────────────┐
│ Load Component   │ (Python function → Docker image)
└──────┬───────────┘
       │ (artifact: DataFrame)
       ↓
┌──────────────────┐
│ Preprocess Comp. │ (handles missing values, scaling)
└──────┬───────────┘
       │ (artifact: cleaned DataFrame)
       ↓
┌──────────────────┐
│ Feature Eng. C.  │ (creates features, correlation analysis)
└──────┬───────────┘
       │ (artifact: feature matrix + metadata)
       ↓
┌──────────────────┐
│ Train Component  │ (model + metrics)
└──────┬───────────┘
       │ (artifacts: model.pkl, metrics.json)
       ↓
┌──────────────┐
│ MinIO Store  │
└──────────────┘
```

### Topics

1. **KFP SDK Fundamentals**
   - Components as Python functions with type hints
   - Function-based vs class-based components
   - Input/output definitions (parameters, artifacts)
   - Container image specifications
   - Dependencies management (requirements.txt)

2. **Building Components for Churn Pipeline**
   - **Load Component**: Read CSV from MinIO, return Pandas DataFrame
   - **Preprocess Component**: Handle missing values, outlier detection, type conversion
   - **Feature Engineering Component**: Create domain features (recency, frequency, monetary value), normalize
   - **Validation Component**: Data quality checks, statistics logging
   - **Train Component**: Fit logistic regression/XGBoost, output model artifact

3. **Pipeline Assembly**
   - Define pipeline DAG using Python SDK
   - Connect components via artifact passing
   - Handle pipeline parameters (train/val/test split ratios)
   - Manage execution order and dependencies

4. **Artifact Management**
   - Artifact types: models, datasets, metrics, visualizations
   - Artifact storage in MinIO buckets
   - Metadata tracking for artifacts (name, type, URI)
   - Artifact lineage across pipeline runs

5. **Compilation & Deployment**
   - Compile pipeline to YAML representation
   - Upload to Kubeflow Pipelines Server
   - Create pipeline runs from dashboard or SDK
   - Parameter injection at runtime

### Pipeline Workflow: Customer Churn

```
Input: customer_churn.csv (10K rows, 12 features)
  ↓
[Load] → DataFrame in memory
  ↓
[Preprocess] → Handle NaN, clip outliers, ensure types
  ↓
[Feature Eng] → Create RFM features, normalize
  ↓
[Validate] → Quality checks, log statistics
  ↓
[Train] → Split into train/val/test, train model, save to MinIO
  ↓
Output: 
  - model.pkl (artifact in MinIO)
  - metrics.json (accuracy, precision, recall, AUC)
  - feature_importance.png (visualization)
```

### Hands-On Exercises

**Exercise 2.1: Create First Component**
- Write `load_data` component that:
  - Loads CSV from MinIO using boto3
  - Returns Pandas DataFrame
  - Includes error handling for missing files
- Build component container image and push to registry
- ✓ Success: Component runs independently, loads 10K row dataset, executes in <30s

**Exercise 2.2: Build Preprocessing Component**
- Write `preprocess_data` component that:
  - Accepts DataFrame artifact as input
  - Handles missing values (strategy: mean/mode/drop)
  - Detects and handles outliers (IQR method)
  - Logs preprocessing statistics (rows removed, imputation counts)
- ✓ Success: Outputs cleaned DataFrame, statistics match expected quality thresholds

**Exercise 2.3: Feature Engineering Component**
- Write `engineer_features` component that:
  - Creates domain features from churn dataset:
    - `recency`: days since last login
    - `frequency`: purchases in last 90 days
    - `monetary`: avg order value
    - `engagement_score`: support tickets + reviews
  - Applies scaling (StandardScaler)
  - Outputs feature matrix + schema metadata
- ✓ Success: Output has correct shape, feature statistics logged, no NaN values

**Exercise 2.4: Assemble Full Pipeline**
- Create `churn_pipeline()` function that:
  - Chains all components in DAG
  - Passes artifacts between steps
  - Adds pipeline parameters: `test_split`, `random_seed`
  - Compiles to YAML
- Run via Kubeflow dashboard
- Monitor execution (all steps should complete in <5 min total)
- ✓ Success: Pipeline runs end-to-end, all artifacts in MinIO, run time <5 min

**Exercise 2.5: Explore Artifacts & Metadata**
- Navigate Kubeflow dashboard → Runs
- Inspect artifacts from completed run
- Download model.pkl from MinIO console
- View metrics.json in dashboard UI
- Access logs for each component step
- ✓ Success: Can download artifacts, view metrics, understand data flow from logs

### Common Pipeline Issues

| Problem | Cause | Solution |
|---------|-------|----------|
| Component fails: "No module named X" | Missing dependency | Add to `requirements.txt`, rebuild Docker image |
| Artifact not found in next step | MinIO credentials wrong | Check KFP config for MinIO access key/secret |
| Component timeout after 10 min | Heavy computation | Increase timeout in component decorator: `@dsl.component(timeout=3600)` |
| Pipeline stuck in "Running" state | Pod evicted/OOM killed | Check pod logs: `kubectl logs <pod> -n kubeflow`, increase memory limits |

### Kubeflow Features Highlighted
- Component versioning via container image tags
- Automatic artifact tracking and metadata
- Lineage visualization (which artifacts flow between steps)
- Caching (re-use outputs of unchanged components)
- Resource limits per component (CPU, memory, GPU)

### Resources
- [KFP Python SDK Docs](https://kubeflow-pipelines.readthedocs.io/)
- [KFP Examples](https://github.com/kubeflow/pipelines/tree/master/samples/v2)
- [Component Best Practices](https://www.kubeflow.org/docs/components/pipelines/sdk/component-development/)

---

## Section 3: Advanced Pipeline Design

### Learning Objectives
- Design complex pipelines with conditional execution and loops
- Manage pipeline parameters and dynamic behavior
- Implement caching for optimization
- Create reusable pipeline templates
- Understand Kubeflow's execution engine

### Core Concepts

**Dynamic Pipelines**: Make pipeline structure responsive to data and parameters
```
┌─────────────────┐
│   Input Data    │
└────────┬────────┘
         │
    ┌────▼────┐
    │ Validate │
    └────┬────┘
         │
    ┌────▼─────────────────────┐
    │ Is data quality OK?       │
    │ YES: continue             │
    │ NO: send alert & exit     │
    └────┬─────────────────────┘
         │
    ┌────▼──────────────────────────┐
    │ Loop: build multiple models   │
    │ - For each algorithm:         │
    │   - Train                     │
    │   - Evaluate                  │
    └────┬───────────────────────────┘
         │
    ┌────▼────────────┐
    │ Compare results │
    │ Select best     │
    └─────────────────┘
```

### Topics

1. **Conditional Execution**
   - Using `dsl.Condition` to branch pipeline execution
   - Example: skip training if data quality too low
   - Example: use fast model for experiments, slow model for production
   - Handling multiple branches that converge

2. **Loops & Loops Over List**
   - `dsl.ParallelFor` for running components multiple times with different parameters
   - Use case: hyperparameter search (try 5 learning rates in parallel)
   - Use case: cross-validation (train on 5 different folds)
   - Use case: A/B testing (train two models in parallel)

3. **Pipeline Parameterization**
   - String, int, float, list parameters
   - Parameter scope: pipeline-level vs component-level
   - Runtime parameter injection
   - Parameter validation and defaults
   - Use case: same pipeline for daily/weekly/monthly retraining with different data windows

4. **Component Caching & Optimization**
   - KFP automatically caches component outputs based on:
     - Component code
     - Input parameters
     - Container image digest
   - Enable/disable caching per component
   - Cache key strategies
   - When to use: expensive data processing, long-running training
   - When to skip: components with side effects, dynamic data sources

5. **Artifact Lineage & Metadata**
   - Kubeflow tracks: which artifact → which component → which run
   - ML Metadata service stores lineage information
   - Query artifact history: "Show all models trained on this dataset"
   - Debug pipeline: "Why did this training metric differ from last run?"
   - Data governance: "Where did this model come from?"

6. **Pipeline Orchestration Patterns**
   - **Sequential**: Task B waits for Task A (default in KFP)
   - **Parallel**: Tasks run simultaneously if independent
   - **Fan-out/Fan-in**: One task spawns many, then collect results
   - **Conditional branching**: Execute task based on previous results

### Example: Advanced Churn Pipeline

```
1. Load Data (always)
   ↓
2. Validate Data Quality
   ├─ If quality < threshold:
   │  ├─ Send alert (email notification)
   │  └─ Exit pipeline
   │
   └─ If quality ≥ threshold:
      ↓
      3. Preprocess Data (cached)
      ↓
      4. Feature Engineering (cached)
      ↓
      5. Split Data (train/val/test)
      ↓
      6. Loop over algorithms: [logistic_regression, xgboost, random_forest]
      ├─ For each: Train model
      ├─ For each: Evaluate on validation set
      ├─ For each: Log metrics
      └─ Collect all trained models
         ↓
      7. Compare & Select Best Model
      ├─ Compare AUC, precision, recall
      └─ Save winning model artifact
         ↓
      8. (Optional) Run explainability analysis on best model
```

### Hands-On Exercises

**Exercise 3.1: Add Data Validation Step**
- Create `validate_data` component that:
  - Checks for minimum row count (threshold: 5000)
  - Checks feature count (threshold: 10+)
  - Checks target variable balance (threshold: not more than 90/10)
  - Returns boolean status + validation report
- Add conditional: if validation fails, skip rest of pipeline and log error
- ✓ Success: Pipeline branches correctly on validation failure, logs informative error message

**Exercise 3.2: Implement Caching**
- Mark `engineer_features` component to use caching
- Run pipeline twice with same input data
- Second run should skip feature engineering (use cached output)
- Verify timing: second run 50%+ faster
- ✓ Success: Cache hit observed, output identical between runs, second execution faster

**Exercise 3.3: Parallel Model Training Loop**
- Create training component `train_model(algorithm, train_data)`
- Create evaluation component `evaluate_model(model, val_data)`
- Use `dsl.ParallelFor` to train 3 algorithms in parallel:
  - Logistic Regression
  - XGBoost
  - Random Forest
- Each algorithm trains independently, evaluations happen in parallel
- ✓ Success: All 3 models train simultaneously, total pipeline time ~same as single model

**Exercise 3.4: Best Model Selection**
- Create `select_best_model` component that:
  - Accepts all 3 trained models + evaluation metrics
  - Compares AUC scores
  - Returns path to best model in MinIO
  - Logs why it was selected
- ✓ Success: Best model correctly identified, selection logic logged, artifact saved

**Exercise 3.5: Query Pipeline Lineage**
- Use Kubeflow dashboard or ML Metadata SDK to:
  - Find all artifacts produced by pipeline runs
  - Trace lineage: specific model → training component → input data
  - Compare two runs: what changed between them?
  - View execution history: how many times was feature engineering cached?
- ✓ Success: Can answer questions about data flow and model provenance

### Advanced Kubeflow Features Showcased
- Dynamic pipeline DAG construction
- Conditional branching with `dsl.Condition`
- Parallel execution with `dsl.ParallelFor`
- Automatic caching based on content
- ML Metadata tracking (lineage, provenance)
- Pipeline versioning (same pipeline definition, different parameters)

### Common Patterns & Gotchas

| Pattern | Example | Gotcha |
|---------|---------|--------|
| Parallel training loop | Train 5 different models simultaneously | Memory: ensure worker nodes have enough memory for all parallel tasks |
| Conditional branches | Run DataProfiler only on new data | Branches must reconnect; can't have orphaned tasks |
| Fan-out/Fan-in | Process each file, then aggregate | Use `dsl.ParallelFor` with step outputs, not manual loops |
| Caching | Skip expensive preprocessing if inputs unchanged | Cache invalidates if component code changes; know your cache keys |

### Resources
- [KFP DSL Docs: Conditions & Loops](https://kubeflow-pipelines.readthedocs.io/)
- [ML Metadata Guide](https://www.kubeflow.org/docs/components/metadata/)
- [KFP Advanced Examples](https://github.com/kubeflow/pipelines/tree/master/samples/v2)
- [Kubeflow Blog: Pipeline Patterns](https://www.kubeflow.org/blog/)

---

## Section 4: Distributed Training at Scale

### Learning Objectives
- Master Kubeflow training operators (PyTorchJob, TFJob)
- Understand distributed training architectures
- Run single-node and multi-node training jobs
- Monitor training with TensorBoard and Prometheus
- Optimize resource utilization and training time

### Core Concepts

**Training Operators**: Native Kubernetes resource types for ML training
```
PyTorchJob/TFJob Architecture:
┌─────────────────────────────────────────┐
│       Kubeflow Training Operator        │
│  (manages job lifecycle)                │
├─────────────────────────────────────────┤
│                                         │
│  Chief Pod (master/rank-0)             │
│  ├─ Initiates communication           │
│  ├─ Logs main training progress       │
│  └─ Responsible for checkpointing     │
│                                         │
│  Worker Pod 1 (rank-1)                │
│  ├─ Parallel data processing          │
│  └─ Synchronized gradient updates    │
│                                         │
│  Worker Pod 2 (rank-2)                │
│  ├─ Parallel data processing          │
│  └─ Synchronized gradient updates    │
│                                         │
│  Parameter Server (optional, TFJob)   │
│  └─ Aggregates gradients              │
│                                         │
└─────────────────────────────────────────┘
```

### Topics

1. **Kubeflow Training Operators**
   - **PyTorchJob**: For PyTorch distributed training
     - Roles: Chief (rank 0), Worker (rank 1+)
     - Uses `torch.distributed` for communication
     - All-reduce operations for gradient synchronization
   - **TFJob**: For TensorFlow distributed training
     - Roles: Chief, Workers, Parameter Servers, Evaluator
     - More flexible for parameter server architecture
   - **Other operators**: MPIJob (for HPC workloads), PaddleJob, XGBoostJob
   - Job lifecycle: created → scheduled → running → success/failure

2. **Data Parallel Training**
   - Each worker loads same model, different data batch
   - Synchronized gradient updates across workers
   - Communication pattern: AllReduce
   - Scalability: linear speedup up to ~16 workers (then communication bottleneck)
   - Use case: training on customer churn dataset with 100GB+ data

3. **Model Parallel Training**
   - Model split across multiple workers (different layers on different devices)
   - For very large models that don't fit on single GPU
   - Uneven compute distribution, harder to tune
   - Use case: Large NLP models, computer vision models with billions of parameters

4. **Job Configuration & Resource Management**
   - Resource requests/limits: CPU cores, memory, GPU count
   - Restart policies: never, on-failure, always
   - Timeout configurations
   - Volume mounts for shared storage
   - Inter-pod communication (service discovery)

5. **Monitoring & Debugging Training**
   - TensorBoard integration: log scalars, histograms, images
   - Prometheus metrics: training loss, accuracy, batch time
   - Pod logs: `kubectl logs <pod-name>`
   - Events: `kubectl describe pytorchjob <job-name>`
   - Training checkpoints: periodic saves to MinIO

6. **Fault Tolerance & Checkpointing**
   - Distributed training can fail on any worker
   - Save checkpoints regularly (every N batches)
   - Resume from checkpoint on failure
   - ElasticJob (future): automatic scaling during training

### Example: Single-Node Training Pipeline Integration

```
KFP Pipeline Step: Train Component
┌──────────────────────────────────────┐
│ Component Input: features.pkl         │
├──────────────────────────────────────┤
│                                      │
│ Create PyTorchJob CRD:              │
│ - 1 Chief pod                       │
│ - Training script: train.py          │
│ - Mount input artifact               │
│ - Mount MinIO volume                │
│ - Resource: 2 CPU, 4GB RAM          │
│                                      │
│ Wait for job completion             │
│ Collect outputs:                    │
│ ├─ model.pth (artifact)            │
│ ├─ metrics.json (artifact)          │
│ └─ training_log.txt (logs)          │
│                                      │
└──────────────────────────────────────┘
Component Output: model.pth
```

### Example: Distributed Training (2 Workers)

```
Training data: 10GB churn dataset
Worker distribution: 5GB per worker
┌──────────────┐         ┌──────────────┐
│  Chief Pod   │         │  Worker Pod  │
│              │◄───────►│              │
│ Rank 0       │  gRPC   │ Rank 1       │
│              │         │              │
│ Loads 5GB    │         │ Loads 5GB    │
│ Trains epoch │         │ Trains epoch │
│ Computes avg │────────►│ Computes avg │
│   gradients  │◄────────│   gradients  │
│              │ AllReduce             │
│ Updates model│         │ Updates model│
│              │         │              │
└──────────────┘         └──────────────┘
Total time: ~40% faster than single-node
```

### Hands-On Exercises

**Exercise 4.1: Single-Node PyTorchJob**
- Create training script `train_churn.py` that:
  - Loads preprocessed features from MinIO
  - Defines PyTorch model (2 hidden layers, 128 units each)
  - Trains with Adam optimizer, binary cross-entropy loss
  - Logs loss every 10 batches
  - Saves trained model to MinIO every epoch
  - Final test accuracy on test set
- Create KFP component that submits PyTorchJob
- Configure resources: 2 CPU cores, 4GB RAM, 30 min timeout
- ✓ Success: Job completes, model saved to MinIO, final accuracy >85%, training <5 min

**Exercise 4.2: Monitor Training with Logs & Metrics**
- Run training job and stream logs: `kubectl logs -f <pod-name> -n kubeflow`
- Extract metrics from logs: training loss per epoch
- Use dashboard to view job status, pod resources, events
- Verify model was saved to MinIO with correct filename
- ✓ Success: Logs show training progress, metrics extracted, job status clear, model in MinIO

**Exercise 4.3: Distributed Training with 2 Workers**
- Modify training script to use `torch.distributed.init_process_group()`
- Create PyTorchJob CRD with:
  - 1 Chief pod
  - 2 Worker pods
  - All with same training script
  - Shared data via MinIO mount
- Launch job: distributed training should use AllReduce for gradient sync
- Compare runtime: should be ~30-40% faster than single node
- ✓ Success: Job runs with 3 pods, all reach same final accuracy, completes 30%+ faster

**Exercise 4.4: Checkpointing & Resume Training**
- Modify training script to:
  - Save checkpoint every 5 batches to MinIO
  - Load from checkpoint if resuming (check for latest ckpt)
  - Log checkpoint frequency and sizes
- Interrupt training job after 10 batches
- Restart same PyTorchJob, verify it loads checkpoint and continues
- ✓ Success: Resumed job starts from batch 10, not batch 0, total time <70% of scratch

**Exercise 4.5: Integrate with KFP Pipeline**
- Create `train_model` component as a KFP step
- Component should:
  - Accept preprocessed data as input artifact
  - Submit PyTorchJob and wait for completion
  - Collect model.pth as output artifact
  - Return metrics.json
- Add to churn pipeline between feature engineering and model evaluation
- Run full pipeline, verify artifact flow from features → training → model
- ✓ Success: Full pipeline end-to-end, training as single unified step, artifacts tracked

### Advanced Training Features

**TensorBoard Integration**
- Log training metrics to TensorBoard event files
- Mount TensorBoard logdir from MinIO
- Access TensorBoard UI via Kubeflow dashboard or port-forward

**Distributed Training Best Practices**
- Batch size: scale linearly with number of workers (x2 workers → x2 batch size)
- Learning rate: may need adjustment for larger effective batch sizes
- Synchronization: full synchronization (all-reduce) vs asynchronous (staleness-tolerant)
- Communication: use fast interconnect if available, watch for network bottlenecks

**Resource Optimization**
- Measure: actual GPU/CPU utilization during training
- Don't over-provision: set requests close to actual usage
- Monitor: watch for OOM kills or evictions
- Scale: add more workers only if training is communication-bottlenecked

### Common Training Issues

| Problem | Cause | Solution |
|---------|-------|----------|
| Distributed job hangs | Worker failed, chief waiting | Check worker pod logs, increase timeout |
| Training much slower with 2 workers | Network bandwidth | Verify inter-pod communication, consider data locality |
| Model.pth not saved | MinIO mount issue or path wrong | Verify MinIO bucket permissions, use absolute paths |
| GPU not detected in pod | Device plugin missing | Check: `kubectl get nodes -o wide`, `nvidia-smi` in pod |
| Job times out at 30 min | Training script slower than expected | Increase timeout or optimize training loop |

### Kubeflow Training Features Highlighted
- Fault tolerance via job restarts
- Automatic environment setup (RANK, WORLD_SIZE, MASTER_ADDR set by operator)
- Service discovery for inter-pod communication
- Resource quota enforcement
- Job history and status tracking

### Resources
- [Kubeflow Training Operators](https://www.kubeflow.org/docs/components/training/)
- [PyTorchJob Docs](https://github.com/kubeflow/training-operator/blob/master/docs/pytorch.md)
- [TFJob Docs](https://github.com/kubeflow/training-operator/blob/master/docs/tensorflow.md)
- [PyTorch Distributed Training](https://pytorch.org/docs/stable/distributed.html)
- [TensorFlow Distributed Training](https://www.tensorflow.org/guide/distributed_training)

---

## Section 5: Hyperparameter Optimization with Katib

### Learning Objectives
- Master hyperparameter optimization (HPO) concepts
- Define search spaces and objectives in Katib
- Compare different search algorithms
- Integrate Katib into production ML workflows
- Optimize expensive training jobs with intelligent search

### Core Concepts

**Hyperparameter Optimization Problem**:
- Manual tuning: Try learning_rate=[0.001, 0.01, 0.1], pick best → 3 trials
- Grid search: 10×10×10 learning rates/batch sizes/regularization → 1000 trials
- Katib: Intelligently sample hyperparameter space, find optima in 50-100 trials

**Katib Search Algorithms**:
```
Random Search          Grid Search           Bayesian Optimization
(uniform random)       (exhaustive)          (probability model)
Points: scattered      Points: regular grid   Points: intelligent
Trials: 100+           Trials: 1000s          Trials: 50-100
Best for: unknown      Best for: known        Best for: expensive
          space               space                   objective
```

### Topics

1. **Hyperparameter Optimization Fundamentals**
   - **Hyperparameters**: Model configuration set before training (learning rate, batch size, dropout, etc.)
   - **Objective metric**: What to optimize (accuracy, AUC, loss)
   - **Search space**: Ranges for each hyperparameter (learning rate: 0.0001-0.1)
   - **Trial**: One full training run with specific hyperparameter values
   - **Experiment**: Coordinated set of trials to find optima

2. **Katib Architecture**
   - Katib controller: orchestrates trials, manages experiment state
   - Trial Suggestion service: recommends hyperparameters (uses search algorithm)
   - Early stopping service: stops unpromising trials early
   - TensorFlow/PyTorch event listener: parses metrics from training logs
   - Experiment CRD: Kubernetes custom resource defining the optimization task

3. **Search Algorithms in Katib**
   - **Random Search**: Sample randomly from search space
     - Simple, parallelizable, baseline comparison
   - **Grid Search**: Systematic coverage of space
     - Exhaustive, finds global optimum in bounded space
     - Computationally expensive for high-dimensional spaces
   - **Bayesian Optimization**: Build probability model, sample intelligently
     - Efficient (fewer trials needed)
     - Balances exploration vs exploitation
     - Good for expensive objectives (training takes hours)

4. **Defining Experiments**
   - Objective: metric name + direction (maximize/minimize)
   - Algorithm: which search strategy to use
   - Parameters: define continuous (min/max), categorical (values), discrete (min/max/step)
   - Trial template: what job to run (PyTorchJob, TFJob, etc.)
   - Metrics collection: how to extract metric from logs or output

5. **Early Stopping Strategies**
   - Stop unpromising trials early (don't wait for full training)
   - Strategies: median rule (stop if below median), success rate
   - Save compute resources: if 100 trials, stop 50% early
   - Trade-off: may miss good hyperparameters that improve late

6. **Integration with ML Workflow**
   - Katib trials write metrics during training
   - Katib aggregates metrics across trials
   - Best hyperparameters → used for final production model
   - Experiment history → reproducibility and audit trail

### Example: Optimizing Churn Model

**Search Space**:
```
learning_rate: continuous [0.001, 0.1]
batch_size: discrete [16, 32, 64, 128]
hidden_dim: discrete [64, 128, 256]
dropout_rate: continuous [0.1, 0.5]
num_layers: discrete [2, 3, 4]
optimizer: categorical [adam, sgd, rmsprop]
```

**Objective**: Maximize AUC on validation set
**Algorithm**: Bayesian Optimization (30 trials with early stopping)

**Expected Results**:
```
Trial 1: learning_rate=0.05, batch_size=32, ... → AUC=0.82
Trial 2: learning_rate=0.02, batch_size=64, ... → AUC=0.79
...
Trial 15: learning_rate=0.03, batch_size=128, ... → AUC=0.87 (best)
Trial 16-30: Katib samples around best region → AUC≤0.87

Best hyperparameters found after 30 trials (vs 5040 with grid search)
```

### Hands-On Exercises

**Exercise 5.1: Create Katib Experiment (Random Search)**
- Define Katib Experiment CRD for churn model with:
  - Objective: maximize AUC
  - Parameters:
    - `learning_rate`: continuous [0.001, 0.1]
    - `batch_size`: discrete [16, 32, 64]
    - `dropout_rate`: continuous [0.1, 0.5]
  - Algorithm: Random (for baseline)
  - Trials: 10 parallel (will complete in ~10 min total)
  - Trial template: PyTorchJob
- ✓ Success: Experiment completes, 10 trials all succeed, best AUC logged

**Exercise 5.2: Analyze Results & Find Best Hyperparameters**
- Query Katib results:
  - View all trial metrics (learning rate vs AUC)
  - Identify best performing trial
  - Extract best hyperparameters
  - Create comparison table: trial_id, hyperparams, AUC
- Use dashboard to visualize: scatter plot (learning_rate vs AUC)
- ✓ Success: Best trial identified, hyperparameters extracted, visualization shows trend

**Exercise 5.3: Bayesian Optimization Experiment**
- Create new Katib Experiment with:
  - Same parameters as Exercise 5.1
  - Algorithm: Bayesian Optimization
  - Trials: 15 (including early stopping)
  - Early stopping: stop trials that fall below 20th percentile
- Run experiment
- Compare: Bayesian vs Random
  - Bayesian should need fewer trials to reach same AUC
  - Bayesian should converge faster to optima region
- ✓ Success: Bayesian experiment completes, achieves better AUC in fewer trials than Random

**Exercise 5.4: Use Best Hyperparameters in Training**
- Extract best hyperparameters from Katib experiment
- Retrain full model using these hyperparameters on full dataset
- Compare final metrics to baseline (default hyperparameters)
- Improvement should be: 3-8% better AUC
- ✓ Success: Final model shows measurable improvement, best params clearly identified

**Exercise 5.5: Integrate Katib into KFP Pipeline**
- Create KFP component that:
  - Submits Katib Experiment
  - Polls for experiment completion
  - Extracts best hyperparameters
  - Returns best params as component output
- Add to churn pipeline:
  - KFP: Load data → Preprocess → Feature Eng → **[NEW] Katib HPO** → Train with best params → Evaluate
- Full pipeline runs HPO as single step
- ✓ Success: HPO runs as pipeline step, best params passed to training step, pipeline completes end-to-end

### Advanced Katib Features

**Custom Metrics Collectors**
- Default: parse logs for metric values
- Custom: extract metrics from REST endpoints (model monitoring)
- Example: collect model inference latency during training

**Multi-Objective Optimization**
- Optimize multiple objectives simultaneously (AUC + inference speed)
- Pareto frontier: set of non-dominated solutions
- Trade-off: find models that are both fast and accurate

**Conditional Parameters**
- Parameter visibility depends on other parameters
- Example: if optimizer=SGD, then momentum (0-0.9), else NA
- Reduces search space automatically

**Experiment Scheduling**
- Scheduled experiments: run optimization weekly/monthly
- Update model hyperparameters as data distribution changes
- Track optimization results over time

### Common Katib Issues

| Problem | Cause | Solution |
|---------|-------|----------|
| Trials fail with "metric not found" | Metric name in YAML differs from training log | Verify metric name matches exactly, check logs: `kubectl logs <trial-pod>` |
| Bayesian optimization converges slowly | Search space too large or metric noisy | Reduce parameter ranges, increase trials, check metric stability |
| All trials have same hyperparameters | Suggestion service not working | Check: `kubectl get pods -n kubeflow \| grep suggestion` |
| Experiment stuck in "running" | Trial job crashed or timeout too short | Increase timeout in trial spec, check trial pod events |

### Kubeflow Features Highlighted
- Distributed hyperparameter tuning across cluster
- Multiple search algorithms with same interface
- Early stopping for resource efficiency
- Automatic metric collection from logs
- Experiment history and reproducibility
- Integration with training operators

### When to Use Katib vs Manual Tuning
| Scenario | Approach | Why |
|----------|----------|-----|
| Training takes 5 min, adjust 2 params | Manual (try 5 combinations) | Not worth automation overhead |
| Training takes 2 hours, optimize 5+ params | Katib (10-20 trials) | Bayesian search more efficient |
| Production: update params monthly | Katib (scheduled experiments) | Automate optimization process |
| Research: explore algorithm space | Katib + custom metrics | Track experiments systematically |

### Resources
- [Katib Documentation](https://www.kubeflow.org/docs/components/katib/)
- [Katib Algorithm Reference](https://github.com/kubeflow/katib/blob/master/docs/proposals/algorithms.md)
- [Bayesian Optimization Explained](https://en.wikipedia.org/wiki/Bayesian_optimization)
- [Optuna Framework (similar to Katib)](https://optuna.org/)

---

## Section 6: Model Serving Essentials with KServe

### Learning Objectives
- Understand model serving patterns and KServe architecture
- Deploy trained models as production inference services
- Make predictions via REST API
- Understand autoscaling and canary deployments
- Monitor served models in production

### Core Concepts

**KServe**: Kubernetes-native model serving platform
```
Churn Model Pipeline                  Production Serving
┌──────────────┐                     ┌──────────────────┐
│ Train Step   │                     │  KServe (InferenceService)
│ Output:      │────────────────────►│  • Load model from MinIO
│ model.pth    │                     │  • Expose REST API
└──────────────┘                     │  • Auto-scale based on traffic
                                     │  • Canary/A/B testing
                                     │  • Model versioning
                                     └──────────────────┘
                                              │
Client: curl -X POST http://service:80       │
  -d {"features": [0.5, 0.3, ...]}           │
                                     ┌─────────┴────────┐
                                     │   Response       │
                                     │ {"prediction": 0.8}
                                     └──────────────────┘
```

### Topics

1. **Model Serving Fundamentals**
   - Why dedicated serving: separate concerns (training vs inference)
   - Benefits: versioning, rollback, monitoring, A/B testing
   - Serving patterns: single model, multi-model, model ensemble
   - Performance vs accuracy trade-offs

2. **KServe Architecture**
   - **InferenceService CRD**: Kubernetes custom resource
   - **Predictor**: Actual model serving component
     - Supported runtimes: SKLearn, XGBoost, PyTorch (TorchServe), TensorFlow, Triton
   - **Transformer** (optional): Preprocessing before prediction
   - **Explainer** (optional): Model explainability service
   - Service discovery: automatic DNS routing

3. **Model Deployment Workflow**
   - Save trained model to MinIO in standard format
   - Create InferenceService YAML pointing to model URI
   - KServe pulls model, loads into memory, exposes gRPC/REST endpoints
   - Automatic Knative Serving integration for autoscaling
   - Traffic routing via Istio

4. **Inference API Basics**
   - REST endpoint: POST `/v1/models/{model_name}:predict`
   - Request format: input features as JSON
   - Response format: predictions + confidence scores
   - Batch prediction: send multiple samples in single request
   - Latency considerations: typical inference <100ms per sample

5. **Production Serving Patterns**
   - **Canary Deployment**: Route small % of traffic to new model, increase gradually
   - **A/B Testing**: Split traffic 50/50 between two models, measure business metrics
   - **Multi-Armed Bandit**: Dynamically route traffic to best-performing model
   - **Shadow Mode**: New model runs alongside current, no traffic sent to it, for validation

6. **Monitoring & Autoscaling**
   - Knative Serving autoscaler: scales based on request rate or target concurrency
   - Metrics: requests/sec, latency (p99), error rate
   - Prometheus integration: export inference metrics
   - Logging: model input/output for debugging
   - Alerting: notify if latency/error rate exceeds threshold

### KServe + Churn Model: Practical Example

```
Production Setup:
1. Save churn model to MinIO: s3://kubeflow-artifacts/models/churn/v1/model.pkl

2. Create InferenceService:
   apiVersion: serving.kserve.io/v1beta1
   kind: InferenceService
   metadata:
     name: churn-predictor
   spec:
     predictor:
       sklearn:
         storageUri: "s3://kubeflow-artifacts/models/churn/v1/"
         resources:
           requests: {cpu: 500m, memory: 1Gi}
     canaryTrafficPercent: 10  # Canary: 10% to new version
```

**Usage**:
```bash
# Make prediction
curl -X POST http://churn-predictor.kubeflow:8080/v1/models/churn-predictor:predict \
  -d '{"instances": [[0.5, 0.8, 1.2, ...]]}'

# Response
{"predictions": [0.92]}  # 92% churn probability
```

### Hands-On Exercises

**Exercise 6.1: Deploy Model as InferenceService**
- Save trained churn model to MinIO: `models/churn-v1/model.pkl`
- Create InferenceService YAML with:
  - SKLearn predictor pointing to MinIO URI
  - Resource limits: 500m CPU, 1GB memory
  - Port 8080 (default)
- Apply InferenceService, wait for pods to be ready
- ✓ Success: Service running, model loaded, REST endpoint accessible

**Exercise 6.2: Test Inference Endpoint**
- Query serving endpoint with test samples:
  ```bash
  curl -X POST http://churn-predictor/v1/models/churn:predict \
    -d '{"instances": [[age, freq, monetary, ...]]}'
  ```
- Make predictions on:
  - 5 individual samples
  - 1 batch request with 10 samples
- Verify responses contain confidence scores
- Check latency: should be <100ms per prediction
- ✓ Success: Get valid predictions, latency <100ms, batch handling works

**Exercise 6.3: Understand Canary Deployments**
- Train an updated churn model (slightly different hyperparameters)
- Save as v2: `models/churn-v2/model.pkl`
- Create new InferenceService with canary setup:
  - Prod (v1) gets 90% traffic
  - Canary (v2) gets 10% traffic
- Send 100 requests, verify ~10 go to v2
- Compare prediction differences between v1 and v2
- ✓ Success: Traffic split correctly, can compare v1 vs v2 predictions on live data

**Exercise 6.4: Integrate Serving into MLOps Pipeline**
- Extend KFP pipeline to include serving:
  - Train step → outputs model.pkl
  - **[NEW] Deploy step**: creates/updates InferenceService
  - Deploy step verifies endpoint is accessible
- Full pipeline: Load → Preprocess → Feature → Train → **Deploy** → Verify
- ✓ Success: Pipeline deploys model to KServe, endpoint functional at end

**Exercise 6.5: Monitor Inference Metrics**
- Deploy model with prometheus metrics enabled
- Send prediction requests for 2 minutes
- Query Prometheus for metrics:
  - Request rate (requests/sec)
  - Prediction latency (p50, p99)
  - Error rate (failures)
- Verify autoscaling: send burst of requests, observe pod scaling
- ✓ Success: Metrics collected, latency <100ms, autoscaling triggered

### KServe Features for Production

**Model Versioning**
- Multiple model versions serve simultaneously
- Route traffic between versions for A/B testing
- Easy rollback: switch traffic back to previous version

**Transformers for Preprocessing**
- Optional preprocessing step before model inference
- Example: normalize features, handle missing values in serving
- Decouples preprocessing from model code

**Explainability**
- Optional explainer service running alongside predictor
- SHAP, LIME, or custom explanations
- Useful for model debugging and transparency

**Multi-Model Services**
- Single service with multiple models
- Route requests based on URL path or header
- Efficient resource sharing

### Common Serving Patterns & Best Practices

| Pattern | Use Case | Trade-off |
|---------|----------|-----------|
| Canary deployment | Validate new model on real traffic | Monitoring overhead |
| Blue-green | Instant rollback to old model | Requires 2x resources temporarily |
| A/B testing | Compare model versions quantitatively | Need statistical rigor |
| Shadow mode | Test without production impact | No user feedback signal |

**Optimization Tips**
- Batch predictions when possible (50-100 samples per request)
- Use smaller models or quantization for low-latency requirements
- Co-locate preprocessing and model for reduced network hops
- Monitor prediction cache hit rates if applicable

### Common Serving Issues

| Problem | Cause | Solution |
|---------|-------|----------|
| InferenceService stuck in "Pending" | Model too large, download timeout | Increase timeout, move model to local persistent volume |
| Predictions return NaN or constant | Model weights not loaded correctly | Verify model format, check model compatibility with runtime |
| High latency (>1s per sample) | Batch size too small or compute-bound | Increase batch size, profile model inference code |
| Out-of-memory errors | Model + input data exceed container memory | Increase memory request in InferenceService spec |

### Kubeflow Features Highlighted
- Seamless integration with Kubeflow Pipelines
- Automatic model registry via MinIO
- Knative Serving for intelligent autoscaling
- Istio-based traffic management
- Multi-tenancy with namespace isolation
- Metrics & monitoring out-of-the-box

### Resources
- [KServe Documentation](https://kserve.github.io/website/)
- [KServe InferenceService Spec](https://kserve.github.io/website/0.11/modelserving/)
- [Knative Serving Autoscaling](https://knative.dev/docs/serving/autoscaling/)
- [Canary Deployments Best Practices](https://en.wikipedia.org/wiki/Canary_deployment)

---

## Section 7: MLOps End-to-End

### Learning Objectives
- Build complete ML systems that operate autonomously
- Automate training, retraining, and model deployment
- Monitor model performance and data drift in production
- Implement best practices for reproducibility and governance
- Manage the full model lifecycle from training through retirement

### Core Concepts

**MLOps Full Lifecycle**:
```
1. Development                2. Production Automation         3. Monitoring & Governance
┌──────────────┐              ┌───────────────────┐            ┌──────────────────┐
│ ML Pipeline  │  Triggers    │ Scheduled         │            │ Model metrics    │
│ (built &     │─────────────►│ Training          │  Deploy   │ Performance      │
│  tested)     │              │ Cron: Daily       │───────────►│ Data drift       │
└──────────────┘              │ or Event-driven   │            │ Triggers Retrain?│
                              │                   │            └──────────────────┘
                              │ Run: Load→Prep→   │
                              │      Train→HPO→   │
                              │      Eval→Serve   │
                              └───────────────────┘
```

### Topics

1. **Complete ML Pipelines: Train + Serve Integration**
   - Single KFP pipeline encompasses: data loading → training → serving
   - Artifact dependencies ensure models are fully trained before serving
   - Rollback capability: if serving fails, revert to previous model
   - Pipeline versioning: track which pipeline version produced which model

2. **Automated Retraining Triggers**
   - **Scheduled (Cron)**: Retrain daily/weekly on fresh data
     - Use case: customer behavior changes daily
     - Kubeflow Runs: schedule via dashboard UI or API
   - **Event-Driven**: Retrain when data quality metric drops
     - Example: accuracy falls below 85% on monitoring, trigger retraining
     - Webhook: external system (data pipeline, monitoring) triggers Kubeflow
   - **Data-Triggered**: Retrain when new data volume exceeds threshold
     - Example: collect 10K new samples, automatically retrain

3. **Model Registry & Versioning**
   - Model artifact versioning: model.pkl tagged with run_id
   - Metadata tracking: which model version for which time period
   - Promotion workflow: train model → validate → promote to production
   - Lineage: trace model back to training data and hyperparameters

4. **Experiment Tracking & Comparison**
   - Log metrics from every training run: accuracy, AUC, loss, latency
   - Kubeflow Metadata Service: stores all run information
   - Compare runs: side-by-side view of hyperparams, metrics, training time
   - Query across runs: "Show all models trained with learning_rate=0.01"
   - Reproducibility: re-run any historical pipeline with exact same parameters

5. **Production Monitoring for Data & Model Drift**
   - **Data Drift**: Input feature distribution changes
     - Monitor: mean/variance of each feature in serving traffic
     - Alert: if feature distribution differs from training data
     - Action: retrain model on new data
   - **Model Drift**: Model performance degradation
     - Ground truth labels arrive with delay (days/weeks)
     - Monitor prediction accuracy once labels available
     - Alert if accuracy drops >5% from baseline
   - **Prediction Drift**: Model's output distribution changes
     - May indicate data drift or new data patterns
     - Track prediction confidence scores, class distributions

6. **Deployment Strategies for Safe Model Updates**
   - **Rolling Update**: Gradually shift traffic from old to new model
     - Start: 0% new, gradually increase (10%, 25%, 50%, 100%)
     - Rollback: if metrics degrade, revert to previous version
   - **Blue-Green**: Two identical production setups
     - Blue (old), Green (new), instant switchover
     - Rollback: switch back to Blue if issues occur
   - **Canary + Monitoring**: Route small % to new, heavy monitoring
     - Catch bugs early on small user subset
     - Most common in Kubeflow + KServe

7. **Best Practices for Reproducibility**
   - **Code**: Version control all training code and pipelines (Git)
   - **Data**: Version input datasets or track their URIs
   - **Hyperparameters**: Log all hyperparams used in training
   - **Environment**: Docker image versioning for exact dependencies
   - **Seeds**: Set random seeds for reproducible splits and initialization
   - **Artifacts**: Store all outputs (models, metrics) with run ID

8. **CI/CD for ML**
   - **Continuous Integration**: 
     - Unit tests: verify components work correctly
     - Integration tests: test full pipeline on sample data
     - Model tests: verify trained model meets performance threshold
   - **Continuous Deployment**:
     - Automated promotion: if tests pass, deploy to production
     - Staged rollout: canary before full production
     - Monitoring: automatic rollback if production metrics degrade

### Example: Churn Model MLOps Workflow

**Day 1: Initial Setup**
```
1. Data arrives in data lake
2. Event: new data file triggers Kubeflow
3. KFP Pipeline runs:
   Load → Preprocess → Feature Eng → Train
   (Katib HPO for best hyperparams)
   → Evaluate (must achieve >85% AUC)
   → Deploy to KServe (canary: 5% traffic)
4. Metrics logged: accuracy, AUC, training time
5. Model versioned: models/churn/v1/model.pkl
```

**Daily: Production Monitoring**
```
1. Serving traffic routed through KServe
2. Prometheus scrapes metrics:
   - Prediction latency (should be <100ms)
   - Prediction distribution (monitor if changing)
   - Request rate, error rate
3. Kubeflow Metadata Service logs all predictions
4. Data drift detection:
   - Daily check: are customer features same distribution as training?
   - If account_age mean dropped 2 years → potential data drift
5. Model performance monitoring (when labels arrive):
   - Compare predictions to actual churn
   - Track accuracy over time
   - Alert if accuracy drops below 82%
```

**Weekly: Automated Retraining**
```
1. Cron job triggers: Sunday 2 AM
2. Same KFP pipeline runs with latest data
3. If new model AUC > old model AUC + 1%:
   - Automatically promote to production (replace v1)
   - Route 5% canary traffic first
   - Monitor for 1 hour
   - If metrics good, shift remaining traffic
4. Metrics compared to previous model:
   - Better accuracy? ✓ Proceed
   - Worse accuracy? ✗ Keep previous model
   - Same? Deploy anyway (minor version bump)
```

**Emergency: Drift Detected**
```
1. Automated check: model accuracy dropped to 78%
2. Alert sent: "Churn model accuracy degradation"
3. Investigation:
   - Check data drift: yes, new customer segment
   - Check serving logs: older model predictions more accurate
4. Immediate action:
   - Roll back model to previous version
   - Notify data team: investigate new customer segment
   - Schedule urgent retraining with new segment data
```

### Hands-On Exercises

**Exercise 7.1: Build Complete Train-Serve Pipeline**
- Extend churn pipeline to include serving:
  - Train → produces model.pkl
  - Eval → metrics.json (must pass threshold test)
  - Deploy → creates/updates KServe InferenceService
  - Verify → test predictions from served model
- Parameters: data_split, learning_rate, batch_size
- ✓ Success: Single pipeline trains model and deploys it, endpoint functional

**Exercise 7.2: Schedule Recurring Retraining**
- Set up Kubeflow recurring run:
  - Trigger: every 6 hours
  - Pipeline: churn pipeline from 7.1
  - Parameters: use data from last 6 hours only (sliding window)
- Run twice to verify scheduling works
- Check: Kubeflow shows 2 runs with different run IDs
- ✓ Success: Recurring runs triggered at correct interval, each with fresh data

**Exercise 7.3: Implement Model Comparison & Promotion**
- Create evaluation component `compare_models` that:
  - Takes: new model + baseline model
  - Compares: accuracy, AUC, inference latency
  - Logic: if new_auc > baseline_auc + 1%, return "promote=true"
  - Logs: detailed comparison metrics
- Add conditional to pipeline:
  - If promote=true: deploy new model to prod
  - If promote=false: keep baseline model
- Run twice with different data/hyperparams
- One should promote, one should not
- ✓ Success: Correct models promoted, metrics justify decision

**Exercise 7.4: Track Experiment Metrics & Compare Runs**
- Run churn pipeline 3 times with different hyperparameters:
  - Run 1: learning_rate=0.01
  - Run 2: learning_rate=0.05
  - Run 3: learning_rate=0.001
- For each run, log: final_accuracy, training_time, model_size
- Use Kubeflow Metadata API/dashboard to:
  - Compare metrics across all 3 runs
  - Identify best run
  - Query: "Show runs where accuracy > 85%"
- ✓ Success: Metrics compared, best run identified, query works

**Exercise 7.5: Simulate Data Drift & Automated Retraining**
- Create "drift simulator":
  - Artificially change test data distribution (e.g., shift feature values)
  - Simulate older model predicting on drifted data (accuracy drops)
- Monitoring component detects accuracy drop
- Automatically trigger retraining pipeline
- New model trained on drifted data improves accuracy
- Log: "Detected drift, retrained, accuracy recovered from 75% to 87%"
- ✓ Success: Drift detection works, automated retraining triggered, recovery verified

**Exercise 7.6: Document Full MLOps Workflow**
- Create MLOps documentation:
  - Architecture diagram: development → production → monitoring
  - Deployment strategy: how models get to production
  - Rollback procedure: how to revert bad models
  - Monitoring checklist: what metrics to watch
  - Retraining schedule: when/why models get retrained
  - SLA: availability, latency, accuracy commitments
- Include: training frequency, expected accuracy, max prediction latency
- ✓ Success: Complete documentation covers all lifecycle stages

### Advanced MLOps Topics

**Model Governance & Compliance**
- Audit trail: who trained what model, when, with what data
- Explainability: understand why model makes predictions
- Bias detection: ensure model isn't discriminatory
- Privacy: handle sensitive customer data (PII)

**Cost Optimization**
- Track: GPU hours, data transfer, storage costs
- Optimize: cheaper models, reduced retraining frequency
- Spot instances: use cheaper compute for training when acceptable

**Multi-Model Systems**
- Ensemble: combine predictions from multiple models
- A/B testing: compare model versions on real users
- Contextual: different models for different customer segments

### Common MLOps Issues

| Problem | Cause | Solution |
|---------|-------|----------|
| Model degrades in production | No monitoring or drift detection | Implement metrics dashboard, set up data drift alerts |
| Can't reproduce old model | Parameters not logged | Use Kubeflow Metadata, version everything (code, data, hyperparams) |
| Retraining too slow | Full pipeline retrained daily | Implement incremental training, cache unchanged steps |
| Production alerts constant noise | Thresholds too sensitive | Tune alert thresholds, require consistency (multiple failures) |
| Hard to roll back bad model | No previous version available | Keep model versioning, tag each serve deployment |

### MLOps Maturity Levels

| Level | Characteristics | Kubeflow Usage |
|-------|-----------------|---|
| **Manual** | Run training scripts manually, deploy model by hand | Dashboard for ad-hoc runs |
| **Basic Automation** | Cron-triggered retraining, manual verification | Scheduled runs, basic pipelines |
| **Advanced** | Event-driven retraining, automated evaluation/promotion | Kubeflow Pipelines + Katib + conditional logic |
| **Mature** | Full monitoring, drift detection, automated rollback | Everything above + ML Metadata, canary deployments |

Your goal: Reach **Advanced** maturity with this course

### Resources
- [Kubeflow Pipelines Documentation](https://www.kubeflow.org/docs/components/pipelines/)
- [MLOps.community](https://mlops.community/)
- [Google Cloud MLOps Best Practices](https://cloud.google.com/architecture/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning)
- [Monitoring ML Models](https://christophergs.com/machine%20learning/2020/03/14/how-to-monitor-machine-learning-models/)

---

## Capstone Project: Production ML System for Customer Churn

### Overview
Integrate everything learned into a single, production-ready ML system that:
- Trains models automatically with optimized hyperparameters
- Deploys models safely with canary rollouts
- Monitors performance and detects data drift
- Retrains automatically when needed

### Requirements

**Must Complete**:
1. End-to-End KFP Pipeline
   - Load customer churn data
   - Preprocess (handle missing values, outliers)
   - Feature engineering (create domain features)
   - Run Katib hyperparameter search (20+ trials)
   - Train best model (PyTorchJob)
   - Evaluate (accuracy >85%, AUC >0.80)
   - Deploy to KServe with inference endpoint

2. Monitoring & Automation
   - Scheduled retraining (daily): automate via Kubeflow runs
   - Log all metrics: accuracy, AUC, training time, model size
   - Track model versions: every trained model has a version
   - Document: create SLA (target accuracy, max latency, availability)

3. Model Comparison & Promotion
   - Compare new model vs baseline
   - Automatic promotion only if AUC improves >1%
   - Keep audit trail: which model served when

4. Inference Testing
   - Test endpoint with 10 sample predictions
   - Measure latency (should be <100ms)
   - Verify batch predictions (10+ samples in one request)

**Nice to Have**:
- Canary deployment (route 10% to new model first)
- Data drift detection (monitor feature distributions)
- Slack notifications when retraining completes
- Dashboard with metrics visualization
- Model explainability (which features matter most?)

### Deliverables

1. **Code Repository**
   - `/pipeline/churn_pipeline.py`: Full KFP pipeline definition
   - `/models/train.py`: Training script for PyTorchJob
   - `/components/`: Individual component definitions (load, preprocess, etc.)
   - `requirements.txt`: All Python dependencies
   - `.gitignore`, README.md

2. **Documentation**
   - `ARCHITECTURE.md`: System design, data flow, components
   - `OPERATIONS.md`: How to schedule runs, monitor, troubleshoot
   - `METRICS.md`: What metrics are tracked, targets, alerts
   - `DEPLOYMENT.md`: How to deploy new models to production

3. **Demonstration**
   - Run full pipeline end-to-end (should complete in <30 min)
   - Show trained model endpoint responding to predictions
   - Show Kubeflow dashboard with run history
   - Show metrics comparison between model versions

### Success Criteria
- Pipeline runs successfully, produces trained model
- Model accuracy ≥85% on test set
- KServe endpoint deployed and responds to inference requests
- Prediction latency <100ms per sample
- Metrics logged and comparable across runs
- Retraining can be scheduled/triggered automatically
- Documentation complete and clear

### Estimated Time
- Implementation: 6-10 hours
- Testing & debugging: 2-4 hours
- Documentation: 2-3 hours
- **Total: 10-17 hours** (weekend project or 2-3 days part-time)

---

## Quick Reference

### Essential kubectl Commands

**Kubeflow Objects**:
```bash
# KFP Pipelines
kubectl get pipelines -n kubeflow                    # List all pipelines
kubectl get pipelinesrun -n kubeflow                # List pipeline runs
kubectl describe pipelinesrun <run-id> -n kubeflow # Run details

# Katib Experiments
kubectl get experiments -n kubeflow                 # List experiments
kubectl describe experiment <exp-name> -n kubeflow # Experiment status
kubectl get trials -n kubeflow                     # List all trials

# Training Jobs
kubectl get pytorchjob -n kubeflow                 # List PyTorch jobs
kubectl get tfjob -n kubeflow                      # List TensorFlow jobs
kubectl describe pytorchjob <job-name> -n kubeflow

# Model Serving
kubectl get isvc -n kubeflow                       # List inference services
kubectl describe isvc <model-name> -n kubeflow     # Service status
```

**Debugging**:
```bash
# Pod logs
kubectl logs <pod-name> -n kubeflow                # Show pod output
kubectl logs -f <pod-name> -n kubeflow             # Follow logs (streaming)
kubectl logs <pod-name> -n kubeflow --tail=100     # Last 100 lines

# Pod details
kubectl get pods -n kubeflow                       # List pods + status
kubectl describe pod <pod-name> -n kubeflow        # Pod events, errors
kubectl get pods -n kubeflow -o wide               # Pods + node assignments

# Events (error messages)
kubectl get events -n kubeflow                     # Recent events
kubectl get events -n kubeflow --sort-by=.metadata.creationTimestamp

# Resource usage
kubectl top pods -n kubeflow                       # CPU/memory per pod
kubectl top nodes                                   # Node resource usage
```

### Port Forwarding (Access Services Locally)

```bash
# Kubeflow Dashboard (access at http://localhost:8080)
kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80

# MinIO Console (access at http://localhost:9001)
kubectl port-forward svc/minio-console -n kubeflow 9001:9090

# Prometheus Metrics (access at http://localhost:9090)
kubectl port-forward -n prometheus svc/prometheus 9090:9090

# TensorBoard (if running)
kubectl port-forward -n kubeflow svc/tensorboard 6006:6006
```

### Troubleshooting Commands

```bash
# Check all Kubeflow components running
kubectl get pods -n kubeflow | grep Running

# Get full error message for failed pod
kubectl describe pod <pod-name> -n kubeflow | grep -A 10 "Events:"

# Debug component/pipeline failures
kubectl logs -f <pipeline-pod-name> -n kubeflow 2>&1 | grep -i error

# Check MinIO access
kubectl exec -it <minio-pod> -n kubeflow -- sh
mc ls kubeflow-artifacts/

# View cluster resources
kubectl cluster-info
kubectl get nodes -o wide
```

### Common Environment Variables

```bash
# Kubeflow namespace
export KUBEFLOW_NAMESPACE=kubeflow
export MY_NAMESPACE=my-ml-project

# MinIO access
export MINIO_ENDPOINT=minio.kubeflow:9000
export MINIO_ACCESS_KEY=minioadmin
export MINIO_SECRET_KEY=minioadmin

# Dashboard
export KUBEFLOW_DASHBOARD=http://localhost:8080
```

### File Locations

```
# KFP compiled pipelines
~/.kfp/                         # KFP SDK cache

# Kubeflow config
~/.kube/config                  # kubectl credentials

# Kubeflow manifests (if self-hosted)
/opt/kubeflow/                  # Kubeflow installation

# MinIO data (persisted)
/var/lib/minio/kubeflow-artifacts/  # MinIO storage (in container)
```

---

## Pre-Course Setup

### System Requirements

**Hardware**:
- [ ] CPU: Intel/AMD processor (4+ cores recommended)
- [ ] RAM: 8GB minimum (12GB+ recommended for distributed training)
- [ ] Disk: 30GB+ free space (Kubeflow + container images take ~20GB)
- [ ] Network: Stable internet connection (will download ~5GB of images)

**Supported OS**:
- [ ] macOS 11+ (Intel or Apple Silicon - some caveats)
- [ ] Ubuntu 20.04 LTS or later
- [ ] Windows 10+ with WSL2 (not recommended for beginners)

**M1/M2 Mac Users**:
- [ ] Know that Docker Desktop handles ARM64 → x86 translation (slower)
- [ ] May see longer startup times for containers
- [ ] Recommendation: use EC2 instance or GCP VM for consistent experience

### Software Prerequisites

**Must Install**:
- [ ] Docker Desktop (or Docker Engine)
  - Verify: `docker --version` (should be 20.10+)
  - Docker daemon must be running
  - Allocate 8GB RAM to Docker (Settings → Resources)
  - Allocate 10GB disk for images
- [ ] kubectl (Kubernetes CLI)
  - Verify: `kubectl version --client`
  - Version 1.21+ recommended
- [ ] k3d (Docker-based Kubernetes)
  - Verify: `k3d --version` (v5.0+)
  - Used to create local k3s clusters

**Recommended**:
- [ ] Git (for version control)
  - Verify: `git --version`
- [ ] Python 3.9+ 
  - Verify: `python3 --version`
  - KFP SDK requires Python 3.9+
- [ ] Pip (Python package manager)
  - Verify: `pip3 --version`

**Optional but Useful**:
- [ ] jq (JSON processor for parsing kubectl output)
- [ ] watch (continuous command monitoring)
- [ ] htop (resource monitoring)

### Pre-Course Python Setup

```bash
# Create virtual environment (optional but recommended)
python3 -m venv kubeflow-env
source kubeflow-env/bin/activate  # On Windows: kubeflow-env\Scripts\activate

# Install Python dependencies
pip install kubeflow kfp torch tensorflow pandas scikit-learn
```

### Verification Checklist

Run these commands to verify everything is set up:

```bash
# Docker
docker run hello-world
# Expected: "Hello from Docker!" message

# Kubernetes
kubectl version --client
# Expected: Client version 1.21+

# k3d
k3d --version
# Expected: k3d version v5.0+

# Python (after installing requirements)
python3 -c "import kfp; print(kfp.__version__)"
# Expected: prints KFP version number
```

### Troubleshooting Pre-Setup Issues

| Issue | Solution |
|-------|----------|
| Docker: "Cannot connect to daemon" | Start Docker Desktop, wait for it to fully boot |
| Docker: "Out of space" | Prune: `docker system prune -a` (removes all unused images) |
| kubectl: "command not found" | Install: `brew install kubectl` (macOS) or download binary |
| k3d: fails to create cluster | Increase Docker memory to 8GB, check disk space >20GB |
| Python: "No module named kfp" | Run: `pip install kubeflow` in virtual environment |

### Initial k3d Cluster Creation

```bash
# Create a k3d cluster for this course (runs once)
k3d cluster create kubeflow-cluster \
  --agents 3 \
  --servers 1 \
  --memory 8G@server \
  --memory 2G@agent*

# Verify cluster is running
kubectl cluster-info
kubectl get nodes
# Expected: 1 server + 3 agent nodes
```

### Next Steps

1. Work through Pre-Course Setup above (15 min)
2. Verify all tools are installed
3. Create k3d cluster (if you plan to do distributed training sections)
4. Proceed to Section 1: Kubeflow Fundamentals

**Time Estimate**: 30-45 min setup, plus 1-2 hours for first-time learning curve

---

*Last updated: 2026-04-05*
