# Kubernetes ConfigMaps

## Introduction
Kubernetes **ConfigMaps** allow you to decouple configuration data from containerized applications. They store non-sensitive configuration details, which can be consumed by containers as environment variables or mounted as configuration files.

## Creating a ConfigMap
You can create a ConfigMap using a YAML file or via `kubectl` command.

### Using a YAML File
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-configmap
data:
  key1: value1
  key2: value2
```
Apply the configuration:
```sh
kubectl apply -f configmap.yaml
```

### Using `kubectl create`
```sh
kubectl create configmap my-configmap --from-literal=key1=value1 --from-literal=key2=value2
```

## Using ConfigMaps
### 1️⃣ As Environment Variables
ConfigMap data can be injected into containers as environment variables.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: my-container
      image: nginx
      env:
        - name: KEY1
          valueFrom:
            configMapKeyRef:
              name: my-configmap
              key: key1
        - name: KEY2
          valueFrom:
            configMapKeyRef:
              name: my-configmap
              key: key2
```
Apply the configuration:
```sh
kubectl apply -f pod.yaml
```

Check if the environment variables are set:
```sh
kubectl exec my-pod -- printenv | grep KEY
```

### 2️⃣ Mounting ConfigMaps as Volumes
ConfigMaps can be mounted as files inside the container.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  volumes:
    - name: config-volume
      configMap:
        name: my-configmap
  containers:
    - name: my-container
      image: nginx
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
```
Apply the configuration:
```sh
kubectl apply -f pod.yaml
```
Verify the mounted files:
```sh
kubectl exec my-pod -- ls /etc/config
```

## Updating a ConfigMap
To update a ConfigMap, modify its YAML definition and reapply it.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-configmap
data:
  key1: new_value1
  key2: updated_value2
```
```sh
kubectl apply -f configmap.yaml
```
**Note:** Pods using environment variables **must be restarted** to see the changes, but mounted volumes update automatically.

## Deleting a ConfigMap
To delete a ConfigMap:
```sh
kubectl delete configmap my-configmap
```

## Understanding Kubernetes ConfigMaps: Why Some Updates Require a Restart 🚀
Have you ever updated a Kubernetes ConfigMap and wondered why:
✅ Mounted volumes update instantly
🔄 But environment variables require a pod restart?

Here's the key difference:

### 1️⃣ ConfigMaps as Environment Variables → Restart Required
When you inject a ConfigMap value as an environment variable, Kubernetes copies the value into the container at startup.

🔹 Problem: Even if you update the ConfigMap, the running pod won’t see the changes because environment variables are static in a process.
🔹 Solution: You must restart the pod to reload the new values.

Example:
```yaml
env:
  - name: DATABASE_URL
    valueFrom:
      configMapKeyRef:
        name: db-config
        key: url
```
If you update `db-config`, the container still holds the old value until a restart.

### 2️⃣ ConfigMaps Mounted as Volumes → No Restart Needed
When you mount a ConfigMap as a volume, Kubernetes creates symbolic links to files inside the container.

🔹 Advantage: If the ConfigMap changes, the file in the container updates instantly, and the app can use the new values without restarting.

Example:
```yaml
volumeMounts:
  - name: config-volume
    mountPath: /etc/config
```
If your app reads `/etc/config/url`, it will always see the latest value from the ConfigMap without a restart.

## Best Practices
- Use environment variables for static configurations (API keys, database URLs), but restart pods after updates.
- Use volume mounts for dynamic configurations (feature flags, log levels) to reflect changes instantly.
- Avoid storing sensitive data in ConfigMaps; use **Secrets** instead.

## Conclusion
ConfigMaps provide a clean way to manage configuration data separately from application logic, ensuring better flexibility and maintainability in Kubernetes environments.

---
#HappyCoding #Kubernetes #DevOps #ConfigMap

Helo