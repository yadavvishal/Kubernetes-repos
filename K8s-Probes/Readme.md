# Implementing Kubernetes liveness, Readiness and Startup probes with nodejs application.

This repository contains Kubernetes configurations to deploy a Node.js application using a **Deployment** and a **Service**.

---

## **Deployment Overview**

The `vishal-nodejs` deployment ensures that:
- Two replicas of the Node.js application run at all times.
- Resource limits and requests are set to optimize performance.
- Health checks (readiness, liveness, and startup probes) ensure application stability.

### **Deployment Details**
- **Image**: `iamdeveloper04/vishal-nodejs`
- **Replicas**: `2`
- **Container Port**: `8080`
- **Resources**:
  - Requests: `128Mi` memory, `512m` CPU
  - Limits: `128Mi` memory, `512m` CPU

### **Health Checks**
- **Readiness Probe**: Ensures the application is ready to receive traffic (`/hello` endpoint)
- **Liveness Probe**: Checks if the application is still running
- **Startup Probe**: Ensures the application has successfully started before other probes begin

---

## **Service Overview**

The `vishal-nodejs` service exposes the Node.js application to external traffic.

### **Service Details**
- **Type**: `NodePort`
- **Exposed Port**: `80`
- **Target Port**: `8080`
- **NodePort**: `30080`

With this configuration, the application can be accessed at:
```
http://<NodeIP>:30080
```

---

## **How to Deploy**

1. Apply the deployment and service files using:
   ```sh
   kubectl apply -f deployment.yaml
   kubectl apply -f service.yaml
   ```
2. Check the running pods or they all are running and restart = 0:
   ```sh
   kubectl get pods
   or
   kubectl get all
   ```
3. Check the exposed service:
   ```sh
   kubectl get svc vishal-nodejs
   ```
4. Access the application:
   ```sh
   curl http://<NodeIP>:30080/hello
   ```

---

## **Scaling Up or Down**
To scale the number of replicas:
```sh
kubectl scale deployment vishal-nodejs --replicas=3
```

---

## **Monitoring Logs**
Check logs of a specific pod:
```sh
kubectl logs <pod-name>
```

---

## **Cleanup**
To delete the deployment and service:
```sh
kubectl delete -f deployment.yaml
kubectl delete -f service.yaml
```

---
