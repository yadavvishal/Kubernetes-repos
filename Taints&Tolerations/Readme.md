# Kubernetes Taints and Tolerations Example

## Overview
This repository demonstrates how to use **Taints and Tolerations** in Kubernetes to control pod scheduling.

## Steps

### 1. Check Available Nodes
```sh
kubectl get nodes
```

### 2. Apply a Taint to a Node
```sh
kubectl taint nodes node1 env=production:NoSchedule
```
Verify the taint:
```sh
kubectl describe node node1 | grep Taints
```
Expected output:
```
Taints: env=production:NoSchedule
```

### 3. Create a Pod Without Toleration
Create a `no-toleration-pod.yaml` file:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: no-toleration-pod
spec:
  containers:
    - name: nginx
      image: nginx
```
Apply the pod:
```sh
kubectl apply -f no-toleration-pod.yaml
```
Check pod status:
```sh
kubectl get pods -o wide
```
Expected: **Pending** (No suitable node found)

### 4. Create a Pod With Toleration
Create a `toleration-pod.yaml` file:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: toleration-pod
spec:
  tolerations:
    - key: "env"
      operator: "Equal"
      value: "production"
      effect: "NoSchedule"
  containers:
    - name: nginx
      image: nginx
```
Apply the pod:
```sh
kubectl apply -f toleration-pod.yaml
```
Verify pod placement:
```sh
kubectl get pods -o wide
```
Expected output:
```
NAME               READY   STATUS    NODE
no-toleration-pod  0/1     Pending   <none>
toleration-pod     1/1     Running   node1
```

### 5. Remove the Taint (Optional)
```sh
kubectl taint nodes node1 env-
```
Verify removal:
```sh
kubectl describe node node1 | grep Taints
```
Expected output:
```
Taints: <none>
```

### Summary
- `node1` was tainted to restrict pod scheduling.
- `no-toleration-pod` **did not** get scheduled on `node1`.
- `toleration-pod` **did** get scheduled due to matching toleration.
- Removing the taint allows all pods to be scheduled again.

## Cleanup
```sh
kubectl delete pod no-toleration-pod toleration-pod
