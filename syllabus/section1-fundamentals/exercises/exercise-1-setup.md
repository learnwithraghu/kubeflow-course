# Exercise 1: Kubeflow Setup Verification

## Objective
Verify your Kubeflow installation is working correctly.

## Tasks

### 1. Check Cluster Connectivity
```bash
kubectl get nodes
```

**Expected Output**: Shows your k3s nodes with STATUS=Ready

### 2. Verify Kubeflow Pods
```bash
kubectl get pods -n kubeflow
```

**Expected**: All pods should be in Running state (may take 5-10 min after install)

### 3. Access Dashboard
```bash
kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80
```

Then open browser: http://localhost:8080

### 4. Create Your Namespace
```bash
# Create a namespace for your experiments
kubectl create namespace kubeflow-demo

# Verify it was created
kubectl get namespace kubeflow-demo
```

## Success Criteria

| Check | Status |
|-------|--------|
| kubectl get nodes shows Ready | [ ] |
| All kubeflow pods Running | [ ] |
| Dashboard accessible | [ ] |
| Namespace created | [ ] |

## Troubleshooting

If pods are not running:
```bash
# Check specific pod details
kubectl describe pod <pod-name> -n kubeflow

# Check pod logs
kubectl logs <pod-name> -n kubeflow
```

## Clean Up
```bash
# Delete test namespace when done
kubectl delete namespace kubeflow-demo
```
