# Kubernetes Resources Practice (`k8-resources`)

This repository contains Kubernetes manifest files and practical commands for managing resources in a Kubernetes cluster (e.g., AWS EKS / Minikube).

---

## 📁 Repository Structure

* [01-namespace.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/k8-resources/01-namespace.yaml) - Defines a dedicated namespace `roboshop` with project labels.
* [02-pod.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/k8-resources/02-pod.yaml) - Creates a standalone `nginx` Pod inside the `roboshop` namespace.

---

## 🚀 Step-by-Step Execution Guide

### 1. Namespaces Management

#### Apply Namespace Manifest (Declarative - Recommended)
```bash
kubectl apply -f 01-namespace.yaml
```

#### Alternative: Create Namespace via CLI (Imperative)
```bash
kubectl create namespace roboshop
```

#### List & Verify Namespaces
```bash
kubectl get namespaces
# or shorthand:
kubectl get ns
```

---

### 2. Pods Management

#### Create / Apply Pod Manifest
```bash
kubectl apply -f 02-pod.yaml
```

#### Check Pods Status in a Specific Namespace
> [!NOTE]
> Since the pod is created in the `roboshop` namespace (`namespace: roboshop`), running `kubectl get pods` without `-n` looks in the `default` namespace and displays `No resources found in default namespace`.

```bash
# Get pods in roboshop namespace
kubectl get pods -n roboshop

# Get pods with detailed info (IP, Node name, Status, etc.)
kubectl get pods -n roboshop -o wide

# Watch pods in real-time
kubectl get pods -n roboshop -w
```

#### Describe & Debug Pods
```bash
# View complete pod details, events, and lifecycle
kubectl describe pod nginx -n roboshop

# View pod logs
kubectl logs nginx -n roboshop

# Stream live pod logs
kubectl logs -f nginx -n roboshop

# Access pod terminal (interactive shell)
kubectl exec -it nginx -n roboshop -- /bin/sh
```

---

### 3. Namespace Scoping & Context Switching (Avoid `-n roboshop` Every Time)

#### ❓ Why does `kubectl describe pod nginx` throw `Error: pods "nginx" not found`?
* Kubernetes scopes all resources by **Namespace**.
* By default, `kubectl` queries the **`default`** namespace unless told otherwise.
* Since our pod is created inside `roboshop`, querying without `-n roboshop` looks in `default` and returns `NotFound`.

#### 💡 Solution 1: Switch Default Context (Built-in)
You can set `roboshop` as your current active namespace for the terminal session:
```bash
kubectl config set-context --current --namespace=roboshop
```
After running this once, you can run all standard commands without `-n roboshop`:
```bash
kubectl get pods
kubectl describe pod nginx
kubectl logs nginx
```

#### 💡 Solution 2: Using `kubens` (Tool used by Instructors)
If you have `kubens` installed on your workstation:
```bash
# Switch active namespace to roboshop
kubens roboshop

# Switch back to default
kubens default
```

---

### 4. Useful Global Commands

#### List Resources Across All Namespaces
```bash
kubectl get pods -A
# or
kubectl get pods --all-namespaces
```

---

### 5. Cleanup / Deletion Commands

#### Delete Resources using Manifests
```bash
# Delete the pod
kubectl delete -f 02-pod.yaml

# Delete the namespace (this automatically deletes all pods inside it)
kubectl delete -f 01-namespace.yaml
```

#### Alternative: Delete by Name
```bash
kubectl delete pod nginx -n roboshop
kubectl delete namespace roboshop
```