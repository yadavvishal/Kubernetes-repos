# Kubernetes Pod Configurations

## Overview
This repository contains Kubernetes pod configuration YAML files demonstrating various scheduling techniques such as node selection, pod affinity, and pod anti-affinity.

## Pod Definitions

### Backend Pod
This pod runs an Nginx container and is scheduled on nodes labeled with `node: backend`.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: backend-pod
  labels:
    name: backend
spec:
  nodeSelector:
    node: backend
  containers:
  - name: backend
    image: nginx
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
```

### Frontend Pod
This pod runs an Nginx container and is scheduled on nodes labeled with `node: frontend`.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend-pod
  labels:
    name: frontend
spec:
  nodeSelector:
    node: frontend
  containers:
  - name: frontend
    image: nginx
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
```

### Pod Affinity Example
This pod uses pod affinity to ensure it is scheduled on the same node as a pod with the label `app: frontend`.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-affinity-pod
spec:
  containers:
    - name: nginx
      image: nginx
      resources:
        limits:
          memory: "128Mi"
          cpu: "500m"
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchExpressions:
              - key: app
                operator: In
                values:
                  - frontend
          topologyKey: "kubernetes.io/hostname"
```

### Pod Anti-Affinity Example
This pod uses pod anti-affinity to ensure it is NOT scheduled on the same node as a pod with the label `app: backend`.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-anti-affinity-pod
spec:
  containers:
    - name: nginx
      image: nginx
      resources:
        limits:
          memory: "128Mi"
          cpu: "500m"
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchExpressions:
              - key: app
                operator: In
                values:
                  - backend
          topologyKey: "kubernetes.io/hostname"
```

## Usage
1. Apply the YAML files using kubectl:
   ```sh
   kubectl apply -f backend-pod.yaml
   kubectl apply -f frontend-pod.yaml
   kubectl apply -f pod-affinity.yaml
   kubectl apply -f pod-anti-affinity.yaml
   ```
2. Verify the pods:
   ```sh
   kubectl get pods -o wide
   ```

## Explanation
- **NodeSelector** ensures that specific pods run on designated nodes.
- **Pod Affinity** ensures that a pod runs on the same node as another specified pod.
- **Pod Anti-Affinity** ensures that a pod does not run on the same node as another specified pod.

