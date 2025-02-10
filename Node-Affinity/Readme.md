# Node Affinity in Kubernetes

## Overview
Node Affinity in Kubernetes allows you to control how Pods are scheduled onto nodes based on labels. This is useful when you want to ensure that specific workloads run on nodes with certain attributes (e.g., SSD storage, GPU, or a particular region).

This example demonstrates both **hard (required)** and **soft (preferred)** node affinity rules in Kubernetes.

---

## Prerequisites
Ensure you have the following before running this example:
- A running Kubernetes cluster
- `kubectl` configured to interact with the cluster

---

## Step 1: Label Your Nodes
Before deploying Pods with Node Affinity, label your nodes appropriately:

```sh
# Label a node with disktype=ssd
kubectl label nodes <node-name-1> disktype=ssd

# Label another node with disktype=hdd
kubectl label nodes <node-name-2> disktype=hdd
```

Verify the labels:
```sh
kubectl get nodes --show-labels
```

---

## Step 2: Deploy a Pod with Hard Node Affinity
This Pod will only run on a node labeled `disktype=ssd`. If no such node is available, it will remain in a **Pending** state.

### **`pod-hard-affinity.yaml`**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-hard-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: disktype
                operator: In
                values:
                  - ssd
  containers:
    - name: nginx-container
      image: nginx
      ports:
        - containerPort: 80
```

Deploy the Pod:
```sh
kubectl apply -f pod-hard-affinity.yaml
```

Check Pod status:
```sh
kubectl get pods -o wide
```

If no SSD-labeled node exists, the Pod will remain **Pending**.

---

## Step 3: Deploy a Pod with Soft Node Affinity
This Pod prefers an SSD node but can be scheduled on other nodes if needed.

### **`pod-soft-affinity.yaml`**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-soft-affinity
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 10
          preference:
            matchExpressions:
              - key: disktype
                operator: In
                values:
                  - ssd
  containers:
    - name: nginx-container
      image: nginx
      ports:
        - containerPort: 80
```

Deploy the Pod:
```sh
kubectl apply -f pod-soft-affinity.yaml
```

Check Pod status:
```sh
kubectl get pods -o wide
```

- If an SSD node is available, the Pod will run there.
- If no SSD node is available, it will run on any available node.

---

## Step 4: Verify Node Placement
To check where the Pods are running:
```sh
kubectl get pods -o wide
```

For detailed Pod information:
```sh
kubectl describe pod pod-with-hard-affinity
kubectl describe pod pod-with-soft-affinity
```

---

## Step 5: Cleanup
If you want to remove the Pods and labels:
```sh
kubectl delete pod pod-with-hard-affinity pod-with-soft-affinity
kubectl label nodes <node-name-1> disktype-
kubectl label nodes <node-name-2> disktype-
```

---

## Summary
| Pod Name | Affinity Type | Behavior |
|----------|--------------|----------|
| `pod-with-hard-affinity` | **Required** (`requiredDuringSchedulingIgnoredDuringExecution`) | Only runs on nodes with `disktype=ssd`. Stays pending otherwise. |
| `pod-with-soft-affinity` | **Preferred** (`preferredDuringSchedulingIgnoredDuringExecution`) | Prefers `disktype=ssd` nodes but can run elsewhere if needed. |

---

## Conclusion
- **Use `requiredDuringSchedulingIgnoredDuringExecution`** when you **must** enforce node placement.
- **Use `preferredDuringSchedulingIgnoredDuringExecution`** when you want Kubernetes to **try** scheduling on specific nodes but allow fallback.

This setup is useful for **optimizing performance**, **resource allocation**, and **ensuring workloads run on specific infrastructure**.

