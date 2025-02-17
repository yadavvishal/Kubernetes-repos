# Node Exporter DaemonSet

This repository contains a Kubernetes DaemonSet configuration for deploying Prometheus Node Exporter on every node in a cluster. The Node Exporter collects and exposes system metrics for monitoring and observability.

## Overview

A **DaemonSet** ensures that a specific pod runs on every node in the Kubernetes cluster. In this case, we use it to deploy **Prometheus Node Exporter**, which gathers system metrics from each node and exposes them on port `9100`.

## Deployment

To deploy the Node Exporter DaemonSet, apply the YAML file:
```sh
kubectl apply -f daemonset.yaml
```
This will ensure that the Node Exporter runs on all nodes.

## DaemonSet Configuration Breakdown

### 1. API Version and Kind
```yaml
apiVersion: apps/v1
kind: DaemonSet
```
- Uses `apps/v1` API version.
- Specifies the kind as `DaemonSet`, ensuring that the pod runs on every node.

### 2. Metadata
```yaml
metadata:
  name: node-exporter
```
- Names the DaemonSet `node-exporter`.

### 3. Selector and Labels
```yaml
spec:
  selector:
    matchLabels:
      app: node-exporter
```
- Ensures the pod is identified with `app: node-exporter`.

### 4. Pod Specification
```yaml
  template:
    metadata:
      labels:
        app: node-exporter
```
- Defines the template for pods that the DaemonSet will create.
- Labels the pod as `app: node-exporter`.

### 5. Container Configuration
```yaml
      containers:
      - name: node-exporter
        image: prom/node-exporter:latest
        args:
          - --path.procfs=/host/proc
          - --path.sysfs=/host/sys
        ports:
        - name: metrics
          containerPort: 9100
```
- Uses the latest `prom/node-exporter` image.
- Exposes port `9100` for Prometheus to scrape metrics.
- Mounts host system directories for metric collection.

### 6. Volume Mounts
```yaml
        volumeMounts:
        - name: procfs
          mountPath: /host/proc
          readOnly: true
        - name: sysfs
          mountPath: /host/sys
          readOnly: true
```
- Mounts `/proc` and `/sys` from the host to read system data.
- Ensures **read-only access** to prevent modifications.

### 7. Volumes
```yaml
      volumes:
      - name: procfs
        hostPath:
          path: /proc
      - name: sysfs
        hostPath:
          path: /sys
```
- Uses `hostPath` to access system information directly from the host.

## Key Considerations for DaemonSets

1. **Use Cases:**
   - Running cluster-wide monitoring agents (e.g., Prometheus Node Exporter, Fluentd, log collection).
   - Ensuring system-critical workloads (e.g., security agents, storage management) run on all nodes.

2. **Scheduling Constraints:**
   - If needed, you can limit DaemonSet pods to specific nodes using `nodeSelector` or `tolerations`.

3. **Security Implications:**
   - Be cautious when mounting host paths, as they grant access to system information.
   - Use `readOnly: true` for sensitive directories.

4. **Scaling & Rolling Updates:**
   - Updating a DaemonSet will automatically roll out changes to all nodes.
   - Using `maxUnavailable` in `updateStrategy` can help prevent downtime.

## Monitoring Metrics
Once deployed, metrics will be available at:
```
http://<node-ip>:9100/metrics
```
You can configure Prometheus to scrape these metrics for monitoring.

## Uninstalling the DaemonSet
To remove the DaemonSet from the cluster:
```sh
kubectl delete -f daemonset.yaml
```

## Conclusion
Using a DaemonSet ensures that Node Exporter runs on every node, making it easy to collect system metrics for monitoring. By properly configuring security, resource limits, and scheduling constraints, you can ensure efficient observability of your Kubernetes cluster.

---

For more information on DaemonSets, visit the [Kubernetes documentation](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/).

