# Kubernetes Resources Practice (`k8-resources`)

This repository contains Kubernetes manifest files and practical command references for deploying, configuring, networking, and managing resources in a Kubernetes cluster (e.g., AWS EKS / Minikube).

---

## 📁 Repository Structure & Manifest Overview

| File | Resource Kind | Purpose / Concept Covered |
| :--- | :--- | :--- |
| [01-namespace.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/k8-resources/01-namespace.yaml) | `Namespace` | Logical cluster isolation with project metadata and labels |
| [02-pod.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/k8-resources/02-pod.yaml) | `Pod` | Single container Pod running NGINX inside `roboshop` namespace |
| [03-multi-container.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/k8-resources/03-multi-container.yaml) | `Pod` | Multi-container Pod (NGINX + AlmaLinux) sharing network & lifecycle |
| [04-labels.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/k8-resources/04-labels.yaml) | `Pod` | Key-Value Labels for grouping, filtering, and selector targeting |
| [05-annotation.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/k8-resources/05-annotation.yaml) | `Pod` | Non-identifying metadata (registry URLs, build info, tools integration) |
| [06-resource.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/k8-resources/06-resource.yaml) | `Pod` | CPU & Memory resource requests (soft limits) and limits (hard caps) |
| [07.env.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/k8-resources/07.env.yaml) | `Pod` | Direct key-value environment variables injected into containers |
| [08-configmap.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/k8-resources/08-configmap.yaml) | `ConfigMap` | Plaintext configuration storage decoupled from application code |
| [09-pod-configmap.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/k8-resources/09-pod-configmap.yaml) | `Pod` | Loading all ConfigMap keys as environment variables (`envFrom`) |
| [10-secretes.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/k8-resources/10-secretes.yaml) | `Secret` | Base64-encoded confidential credentials storage |
| [11-pod-secretes.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/k8-resources/11-pod-secretes.yaml) | `Pod` | Injecting Kubernetes Secrets as container environment variables (`secretRef`) |
| [12-service.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/k8-resources/12-service.yaml) | `Service` | Internal L4 networking and load balancing (`ClusterIP`) using label selectors |

---

## 🚀 Step-by-Step Execution & Commands Guide

### 1. Namespaces (`01-namespace.yaml`)
```bash
# Create / Apply Namespace
kubectl apply -f 01-namespace.yaml

# List Namespaces
kubectl get namespaces
# or shorthand:
kubectl get ns
```

---

### 2. Standalone & Multi-Container Pods (`02-pod.yaml`, `03-multi-container.yaml`)
```bash
# Apply Standalone Pod
kubectl apply -f 02-pod.yaml

# Apply Multi-Container Pod
kubectl apply -f 03-multi-container.yaml

# List Pods in roboshop or default namespace
kubectl get pods -n roboshop
kubectl get pods -o wide

# Access specific container in a multi-container pod:
kubectl exec -it multi-container -c nginx -- /bin/sh
kubectl exec -it multi-container -c almalinux -- bash
```

---

### 3. Labels & Selectors (`04-labels.yaml`)
```bash
# Apply Pod with Labels
kubectl apply -f 04-labels.yaml

# Filter Pods using Label Selectors
kubectl get pods -l project=roboshop
kubectl get pods -l environment=dev,component=frontend
kubectl get pods --show-labels
```

---

### 4. Annotations (`05-annotation.yaml`)
```bash
# Apply Pod with Annotations
kubectl apply -f 05-annotation.yaml

# View annotations attached to pod
kubectl describe pod annotations | grep -A 5 Annotations
```

---

### 5. Resource Requests & Limits (`06-resource.yaml`)
* **Requests (Soft Limit):** Minimum guaranteed compute resources reserved on the node for the container (`cpu: 100m` = 0.1 vCPU, `memory: 128Mi`).
* **Limits (Hard Limit):** Maximum compute resources the container is allowed to consume (`cpu: 200m`, `memory: 256Mi`). If memory exceeds the limit, container is OOMKilled.
```bash
# Apply Resource-constrained Pod
kubectl apply -f 06-resource.yaml

# Inspect allocated requests & limits
kubectl describe pod resources-demo | grep -A 6 Limits
```

---

### 6. Environment Variables (`07.env.yaml`)
```bash
# Apply Pod with direct Environment Variables
kubectl apply -f 07.env.yaml

# Verify Environment Variables inside container
kubectl exec -it env-demo -- env | grep -E "project|course"
```

---

### 7. ConfigMaps (`08-configmap.yaml`, `09-pod-configmap.yaml`)
```bash
# Create ConfigMap
kubectl apply -f 08-configmap.yaml

# View ConfigMap Data
kubectl get configmap nginx-config -o yaml

# Apply Pod consuming ConfigMap
kubectl apply -f 09-pod-configmap.yaml

# Verify injected environment variables
kubectl exec -it pod-configmap-demo -- env | grep -E "course|trainer|duration"
```

---

### 8. Secrets (`10-secretes.yaml`, `11-pod-secretes.yaml`)
* **Note:** Secrets data values must be Base64 encoded (`echo -n "admin" | base64`).
```bash
# Create Secret
kubectl apply -f 10-secretes.yaml

# View Secret Metadata (values are hidden)
kubectl get secret nginx-secret -o yaml

# Apply Pod consuming Secret
kubectl apply -f 11-pod-secretes.yaml

# Verify decrypted secrets inside container environment
kubectl exec -it pod-secretes-demo -- env | grep -E "username|password"
```

---

### 9. Services & Networking (`12-service.yaml`)

#### Core Networking Concepts
* **Service:** Provides a stable, persistent Virtual IP (`ClusterIP`) and DNS name for ephemeral Pods selected via `labels`.
* **Port Mapping:**
  * `port: 80` $\rightarrow$ The port exposed by the Service internally within the cluster.
  * `targetPort: 80` $\rightarrow$ The actual port on which the container application is listening.
* **Service Types:**
  1. **`ClusterIP` (Default):** Accessible **only inside** the cluster (for microservice-to-microservice communication like Frontend to Backend).
  2. **`NodePort`:** Exposes the service on a static high port (`30000-32767`) on each Worker Node's IP (`External IP:NodePort`).
  3. **`LoadBalancer`:** Automatically provisions a Cloud Provider Load Balancer (e.g., AWS Classic/NLB) with a public external endpoint.

```bash
# Apply Service
kubectl apply -f 12-service.yaml

# Check Service details and assigned ClusterIP / Endpoints
kubectl get svc
kubectl get endpoints nginx
kubectl describe svc nginx
```

---

## 🛠️ Troubleshooting & Best Practices

### Namespace Scoping & Context Switching
If `kubectl describe pod <name>` returns `NotFound`, ensure you are querying the correct namespace:
```bash
# Check across all namespaces
kubectl get pods -A

# Switch active namespace permanently for current session:
kubectl config set-context --current --namespace=roboshop
```

### Cleanup All Resources
```bash
# Delete all resources created in this repository:
kubectl delete -f .
```