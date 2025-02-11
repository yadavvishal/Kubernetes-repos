# Node Anti-Affinity Pod

## Overview
This Kubernetes pod configuration ensures that the pod is **not scheduled** on nodes labeled with `disktype=ssd`. It uses **node affinity rules** to enforce this constraint.

## Features
- **Node Anti-Affinity**: Prevents scheduling on nodes with `disktype=ssd`.
- **Resource Requests & Limits**: Defines CPU and memory constraints to prevent excessive resource usage.
- **Simple Pod Specification**: Deploys an Nginx container with specified resource limits.

## YAML Configuration
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: node-anti-affinity-pod
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      limits:
        cpu: "500m"
        memory: "256Mi"
      requests:
        cpu: "250m"
        memory: "128Mi"
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: disktype
                operator: NotIn
                values:
                  - ssd
```

## How It Works
1. **Node Affinity Configuration**:
   - The `nodeAffinity` rule specifies that the pod **must not be scheduled** on nodes where the label `disktype=ssd` exists.
   - The `operator: NotIn` condition ensures exclusion from SSD nodes.

2. **Resource Limits**:
   - CPU limits: `500m` (0.5 vCPU)
   - Memory limits: `256Mi` (256 MB RAM)
   - Requests ensure the pod gets the minimum required resources (`250m` CPU, `128Mi` RAM).

## Deployment Instructions
### **Step 1: Apply the Pod Configuration**
Run the following command to create the pod:
```sh
kubectl apply -f node-anti-affinity-pod.yaml
```

### **Step 2: Verify Pod Placement**
Check where the pod is scheduled:
```sh
kubectl get pods -o wide
```
Ensure the pod is **not running on SSD nodes**.

### **Step 3: Check Node Labels**
If the pod is in `Pending` state, verify if the cluster has nodes without `disktype=ssd`:
```sh
kubectl get nodes --show-labels
```
If necessary, label a node to ensure proper scheduling:
```sh
kubectl label nodes <node-name> disktype=hdd
```

## Notes
- If no suitable nodes are available, the pod will remain in a `Pending` state.
- Modify the `disktype` label in your node configurations if needed.

## Cleanup
To delete the pod, run:
```sh
kubectl delete pod node-anti-affinity-pod
```

---
This setup ensures that pods are **not scheduled on SSD nodes**, improving workload distribution and node resource management. 🚀

