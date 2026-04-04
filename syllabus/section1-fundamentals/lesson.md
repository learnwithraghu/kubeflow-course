# Section 1: Kubeflow Fundamentals

## Lesson 1.1: What is Kubeflow?

### Learning Goals
- Define Kubeflow and understand its purpose
- Compare Kubeflow with other ML platforms
- Identify key benefits of using Kubeflow

### What is Kubeflow?

Kubeflow is an **open-source ML platform** built on Kubernetes that provides tools for building, deploying, and managing ML workflows.

### Why Kubeflow?

| Benefit | Description |
|---------|-------------|
| **Portability** | Run anywhere Kubernetes runs (local, cloud, edge) |
| **Scalability** | Scale ML workloads automatically |
| **Experimentation** | Easy to try different approaches |
| **Reproducibility** | Track experiments and versions |

### Key Use Cases

1. **End-to-end ML pipelines** - From data prep to model serving
2. **Hyperparameter tuning** - Automated search for best parameters
3. **Distributed training** - Scale training across many nodes
4. **Model serving** - Deploy models as REST APIs

### Kubeflow vs Other Platforms

| Platform | Best For | Kubernetes Required? |
|----------|----------|---------------------|
| Kubeflow | Enterprise ML on K8s | Yes |
| MLflow | Experiment tracking | No |
| Airflow | Data pipelines | No |
| SageMaker | AWS-managed ML | No (managed) |

### Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│                  Kubeflow                          │
├─────────────────────────────────────────────────────┤
│  User Interface (Dashboard, JupyterHub)            │
├─────────────────────────────────────────────────────┤
│  ML Tools (Pipelines, Katib, KServe, Training)    │
├─────────────────────────────────────────────────────┤
│  Kubernetes Infrastructure                          │
└─────────────────────────────────────────────────────┘
```

### Core Components

1. **Kubeflow Pipelines (KFP)** - Build ML pipelines
2. **Katib** - Hyperparameter optimization
3. **KServe** - Model serving and inference
4. **Training Operators** - TFJob, PyTorchJob, MPIJob
5. **Central Dashboard** - Web UI for Kubeflow
6. **JupyterHub** - Notebook servers for experimentation

### Your First Interaction

After installing Kubeflow, you'll access the **Central Dashboard** at:
```
http://localhost:8080
```

The dashboard provides:
- Pipeline list and run history
- Katib experiments
- Deployed models
- Notebook servers
- Namespace management

### Summary

- Kubeflow = ML platform on Kubernetes
- Provides portability, scalability, and experiment tracking
- Core components: KFP, Katib, KServe, Training Operators
- Access via Central Dashboard

---

## Lesson 1.2: Installing Kubeflow on k3s

### Learning Goals
- Install k3s (lightweight Kubernetes)
- Deploy Kubeflow using k3d
- Verify installation and access dashboard

### Prerequisites
- Docker installed and running
- 8GB+ RAM available
- 20GB+ disk space

### Step 1: Install k3s

```bash
# macOS (using Homebrew)
brew install k3d

# Or download binary directly
curl -s https://raw.githubusercontent.com/k3s-io/k3s/master/install.sh | sh
```

### Step 2: Create k3s Cluster

```bash
# Create a 3-node cluster (recommended for Kubeflow)
k3d cluster create kubeflow --agents 3

# Verify cluster
kubectl cluster-info
```

### Step 3: Install Kubeflow

We use **k3d** with the Kubeflow manifests:

```bash
# Add Kubeflow manifests repo
git clone https://github.com/kubeflow/manifests.git
cd manifests

# Install Kubeflow (this takes 10-15 minutes)
while ! kustomize build example | kubectl apply -f -; do echo "Retrying..."; sleep 10; done
```

### Alternative: Using Kind

```bash
# Install kind if not installed
brew install kind

# Create cluster
kind create cluster --name kubeflow

# Install Kubeflow
kubectl apply -k github.com/kubeflow/manifests?ref=master
```

### Step 4: Verify Installation

```bash
# Check all Kubeflow pods are running
kubectl get pods -n kubeflow

# Should show pods like:
# - ml-pipeline-xxx
# - katib-xxx
# - kserve-xxx
```

### Step 5: Access Dashboard

```bash
# Port forward to access Kubeflow UI
kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80

# Open browser: http://localhost:8080
```

### Troubleshooting

| Issue | Solution |
|-------|----------|
| Pods stuck in Pending | Increase Docker resources |
| Image pull errors | Check internet connection |
| Port conflict | Use different port (8081:80) |

### Quick Cleanup

```bash
# Delete cluster when done
k3d cluster delete kubeflow

# Or keep for later use
# Cluster persists until deleted
```

### Exercise Checklist

- [ ] Install k3d
- [ ] Create k3s cluster
- [ ] Deploy Kubeflow manifests
- [ ] Wait for all pods to be Running
- [ ] Access Kubeflow dashboard
- [ ] Explore the UI

---

## Lesson 1.3: Kubeflow Dashboard Tour

### Learning Goals
- Navigate the Kubeflow dashboard
- Understand the main sections
- Identify key functionality

### Dashboard Sections

#### 1. Home / Dashboard
- Overview of Kubeflow components
- Quick links to recent runs
- System status

#### 2. Pipelines
- List of uploaded pipelines
- Run history
- Pipeline experiments

#### 3. Runs
- Individual run executions
- Status (running, succeeded, failed)
- Logs and outputs

#### 4. Experiments (KFP)
- Group runs by experiment
- Compare runs
- Filter and search

#### 5. Katib
- Hyperparameter experiments
- Best results
- Search spaces

#### 6. Models (KServe)
- Deployed inference services
- Endpoint status
- Request metrics

#### 7. Notebooks
- Jupyter notebook servers
- Create new notebooks
- Access via JupyterLab

### Quick Actions

```bash
# Create a namespace for your project
kubectl create namespace student-project

# List all namespaces
kubectl get namespaces

# Switch to your namespace in the UI
```

### Practice Tasks

1. Navigate to **Pipelines** section - see if any pre-loaded pipelines exist
2. Click on **Katib** - understand experiment structure
3. Go to **Models** - see deployed inference services
4. Create a **test namespace** via kubectl

---

## Lesson 1.4: Core Concepts Review

### Key Terms

| Term | Definition |
|------|------------|
| **Pipeline** | Series of ML tasks connected as a DAG |
| **Component** | Single task in a pipeline (container) |
| **Run** | Single execution of a pipeline |
| **Experiment** | Group of runs for comparison |
| **Artifact** | Output from a component (model, dataset) |
| **Katib** | Kubeflow's HPO system |
| **KServe** | Kubeflow's model serving |

### What You'll Learn Next

- **Section 2**: Build your first pipeline with KFP
- **Section 3**: Manage data and artifacts
- **Section 4**: Train models on Kubernetes
- **Section 5**: Tune hyperparameters with Katib
- **Section 6**: Serve models with KServe
- **Section 7**: Automate everything

### Summary

- Kubeflow is installed on k3s for local development
- Dashboard provides access to all components
- Core concepts: Pipelines, Components, Runs, Experiments, Artifacts
- Next: Build your first ML pipeline

---

## Exercise 1: Setup Verification

### Objective
Verify your Kubeflow installation is working correctly.

### Tasks

1. **Check cluster connectivity**
   ```bash
   kubectl get nodes
   # Should show your k3s nodes
   ```

2. **Verify Kubeflow pods**
   ```bash
   kubectl get pods -n kubeflow
   # All pods should be Running
   ```

3. **Access dashboard**
   - Open http://localhost:8080
   - Verify login page appears

4. **Create test namespace**
   ```bash
   kubectl create namespace test-$USER
   ```

### Success Criteria
- [ ] `kubectl get nodes` shows running nodes
- [ ] All kubeflow namespace pods are Running
- [ ] Dashboard accessible in browser
- [ ] Test namespace created successfully

### Next Section Preview
In Section 2, you'll build a simple 3-step pipeline:
```
Download Data → Preprocess → Train Model
```
