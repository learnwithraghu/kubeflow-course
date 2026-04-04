# Demo 1.1: First Look at Kubeflow

## Overview
This demo shows Kubeflow's main features through the dashboard.

## Steps

### 1. Start Kubeflow Dashboard
```bash
kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80
```

### 2. Navigate Dashboard
Open http://localhost:8080 and explore:

| Section | What to Look For |
|---------|------------------|
| Home | System status, recent activity |
| Pipelines | Existing pipelines, run history |
| Katib | HPO experiments |
| Models | Deployed inference services |

### 3. Check Kubeflow Version
```bash
# Check installed version
kubectl get deployment -n kubeflow -o wide
```

## Expected Observations

1. **Pipelines section**: Should show Kubeflow Pipelines UI
2. **Katib section**: Can create HPO experiments
3. **Models section**: Can deploy InferenceServices

## Key Takeaways

- Kubeflow provides unified UI for all ML tasks
- All components accessible from central dashboard
- Namespace isolation keeps projects separate
