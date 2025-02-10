# Connecting Pods to a Kubernetes Service Using Labels and Selectors

## Create a Pod with Labels
First, we’ll create a pod and add labels to it so that Kubernetes can identify it using the below YAML file.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: demo
    environment: production
spec:
  containers:
  - name: nginx
    image: nginx-latest
```

### Understanding the YAML Manifest
- A Pod named **my-pod** is created.
- It includes two labels:
  - `app: demo` identifies it as part of the "demo" application.
  - `environment: production` categorizes it under the production environment.
- The Pod runs an **Nginx container** using the default Nginx image.

Save the above configuration in a YAML file called `my-pod.yaml` and use the below command to create a pod with the labels and spec:

```sh
kubectl apply -f my-pod.yaml
```

Check your Pod and its labels:

```sh
kubectl get pods --show-labels
```

You’ll see your Pod listed with its labels.

## Create a Service to Connect the Pod
Now, let’s create a Service to expose the Pod. We’ll use a selector in the Service to match the Pod’s label. This will allow our Service to send traffic to our Pod.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: demo    # Looks for Pods with label "app: demo"
    environment: production # Looks for Pods with label "environment: production"
  ports:
    - port: 80
  type: ClusterIP
```

### Understanding the YAML Manifest
- A **Service** named `my-service` is created.
- **Selector**: The Service targets Pods with both `app: demo` and `environment: production` labels.
- **Ports**: The Service listens on **port 80** and forwards traffic to Pods matching the selector.
- **Type**: The `ClusterIP` type makes the Service accessible **within** the Kubernetes cluster using an internal IP.

This Service ensures only Pods with the right labels receive traffic. The Service makes use of the selector to ensure that the correct set of Pods are being exposed.

Save the above configuration in a YAML file called `my-service.yaml` and use the below command to create the Service:

```sh
kubectl apply -f my-service.yaml
```

Check that everything is connected:

```sh
kubectl get pods --show-labels
```

The command lists all Pods in the current namespace and displays their associated labels. So, you’ll see the labels `app: demo` and `environment: production` associated with the Pod.

## Testing the Connection
To test the Service locally, run the following command:

```sh
kubectl port-forward service/my-service 8080:80
```

This command connects **port 80** of `my-service` to **port 8080** on your computer.

You’ll see output like this:

```
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

Once you've port-forwarded the Service to your local machine, run the command below in a new terminal:

```sh
curl localhost:8080
```

If you receive the **Nginx response**, it means that the Service is successfully running and accessible locally. This confirms that the port-forwarding is working as expected, and your Service is set up properly to handle requests.

