# Demo 1: Kubeflow Installation

## Overview
This demo covers installing Kubeflow on a Kubernetes cluster.

## Prerequisites
- Kubernetes cluster (minikube, kind, or cloud)
- kubectl configured
- kustomize installed

## Steps

### 1. Download Kubeflow Manifests
```bash
# Clone Kubeflow manifests
git clone https://github.com/kubeflow/manifests.git
cd manifests
```

### 2. Install Kubeflow
```bash
# Install with kustomize
kubectl apply -k manifests/kustomize/
```

### 3. Wait for Installation
```bash
# Check pods
kubectl get pods -n kubeflow
```

### 4. Verify Installation
```bash
# Check deployments
kubectl get deployment -n kubeflow
```

## Expected Observations

1. **Pods**: All kubeflow pods should be running
2. **Services**: Istio gateway and other services available

## Key Takeaways

- Kubeflow installs as Kubernetes manifests
- Installation takes several minutes
- All components deploy to kubeflow namespace
