# 🚀 Complete Kubernetes (K8s) Master Learnings & Interview Guide

ఈ డాక్యుమెంట్‌లో మనం నేర్చుకున్న అన్ని Kubernetes కాన్సెప్ట్స్, AWS EKS ఆర్కిటెక్చర్, మ్యానిఫెస్ట్ ఫైల్స్, నెట్‌వర్కింగ్ ఫ్లోస్, కాస్ట్ మేనేజ్‌మెంట్ మరియు ఇంటర్వ్యూ ప్రిపరేషన్ పాయింట్లు చాలా సులభంగా (తెలుగు + సింపుల్ ఇంగ్లీష్) పొందుపరచబడ్డాయి.

---

## 📑 విషయ సూచిక (Table of Contents)
1. [Kubernetes vs Traditional EC2 Deployments](#-1-kubernetes-vs-traditional-ec2-deployments)
2. [AWS EKS & Cost Optimization (Spot Instances)](#-2-aws-eks--cost-optimization-spot-instances)
3. [Namespaces & Context Switching (Imperative vs Declarative)](#-3-namespaces--context-switching)
4. [Pods & Multi-Container Pods (Sidecar Pattern)](#-4-pods--multi-container-pods)
5. [Labels vs Annotations](#-5-labels-vs-annotations)
6. [Resource Management: Requests vs Limits](#-6-resource-management-requests-vs-limits)
7. [Configuration Decoupling: Env, ConfigMaps & Secrets](#-7-configuration-decoupling-env-configmaps--secrets)
8. [Kubernetes Services & Networking (ClusterIP vs NodePort vs LoadBalancer)](#-8-kubernetes-services--networking)
9. [End-to-End Client-to-Service Communication Flow](#-9-end-to-end-client-to-service-communication-flow)
10. [ReplicaSets (Self-Healing, Scaling & HA)](#-10-replicasets-self-healing-scaling--ha)
11. [Top Real-World Debugging Errors & Solutions](#-11-top-real-world-debugging-errors--solutions)
12. [Must-Know Kubernetes Interview Q&A](#-12-must-know-kubernetes-interview-qa)

---

## 📌 1. Kubernetes vs Traditional EC2 Deployments

* **సాంప్రదాయ పద్ధతి (Individual VMs/EC2):**
  * Frontend (Angular), Backend (Node), Cart (Java), AI (Python) లకు విడివిడిగా 4 నుండి 8 సర్వర్లు పెట్టాలి.
  * సర్వర్లలో CPU/RAM కేవలం 10% - 20% మాత్రమే వాడబడుతుంది $\rightarrow$ **రిసోర్స్ వృథా & ఎక్కువ ఖర్చు**.
* **Kubernetes పద్ధతి:**
  * అన్ని సర్వీసులను కంటైనర్లుగా మార్చి కేవలం **2 లేదా 3 షేర్డ్ వర్కర్ నోడ్స్** లో ప్యాక్ చేస్తాం (Resource Packing).
  * CPU/RAM 70% - 80% వరకు పూర్తిగా వినియోగించబడుతుంది.
  * ఆటో-స్కేలింగ్ మరియు స్పాట్ ఇన్స్టాన్సెస్ వల్ల క్లౌడ్ బిల్లు **60% నుండి 80% వరకు తగ్గుతుంది**.

---

## 📌 2. AWS EKS & Cost Optimization (Spot Instances)

### 🔹 EKS Control Plane ఖర్చు
* Amazon EKS Control Plane మేనేజ్‌మెంట్ ఫీ: **$0.10/hour** (24/7 రన్ అయితే నెలకు దాదాపు **~$73**).
* ఇది కేవలం API Server & etcd మేనేజ్ చేసేందుకు AWS తీసుకునే ఫీజు. వర్కర్ నోడ్స్ (EC2) ఖర్చు అదనం.

### 🔹 Spot Instances లాభం
* [eks.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/eksctl/eks.yaml) లో `spot: true` పెట్టి మల్టిపుల్ ఇన్స్టాన్స్ టైప్స్ ఇవ్వడం వల్ల:
  ```yaml
  instanceTypes: ["c3.large","c4.large","c5.large","c5d.large","c5n.large","c5a.large"]
  spot: true
  ```
  * On-Demand ధరతో పోలిస్తే **70-80% తక్కువ ఖర్చుతో** వర్కర్ నోడ్స్ లభిస్తాయి.

---

## 📌 3. Namespaces & Context Switching

* **Namespace ఉద్దేశ్యం:** క్లస్టర్ లోపల వర్చువల్ ఐసోలేషన్ (ఉదా: `roboshop`, `dev`, `prod`).

### 🔹 Imperative (`create`) vs Declarative (`apply -f`)
| అంశం | `kubectl create namespace roboshop` | `kubectl apply -f 01-namespace.yaml` |
| :--- | :--- | :--- |
| **విధానం** | Imperative (Ad-hoc కమాండ్) | Declarative (కోడ్ ఆధారిత) |
| **Re-run చేసినప్పుడు** | `AlreadyExists` Error ఇస్తుంది | Safe (`configured` లేదా `unchanged`) |
| **Labels / Metadata** | కేవలం ఖాళీ నేమ్‌స్పేస్ వస్తుంది | ఫైల్‌లోని లేబుల్స్ అన్నీ సెట్ అవుతాయి |
| **ఉపయోగం** | క్విక్ టెస్టింగ్ కోసం | **CI/CD & ప్రొడక్షన్ కోసం Recommended** |

### 🔹 నేమ్‌స్పేస్ స్కోపింగ్ (NotFound Error పరిష్కారం)
* `kubectl` ఎప్పుడూ డిఫాల్ట్‌గా `default` నేమ్‌స్పేస్‌లోనే వెతుకుతుంది.
* వేరే నేమ్‌స్పేస్‌లోని రిసోర్స్ చూడాలంటే `-n <namespace>` ఇవ్వాలి.
* ప్రతిసారీ `-n` టైప్ చేయకుండా డిఫాల్ట్ కాంటెక్స్ట్ మార్చుకోవడానికి:
  ```bash
  kubectl config set-context --current --namespace=roboshop
  # లేదా
  kubens roboshop
  ```

---

## 📌 4. Pods & Multi-Container Pods

* **Pod:** Kubernetes లో అతి చిన్న డెప్లాయ్‌మెంట్ యూనిట్.
* **Multi-Container Pod (Sidecar Pattern):**
  * [03-multi-container.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/k8-resources/03-multi-container.yaml) లో `nginx` మరియు `almalinux` రెండు కంటైనర్లు ఒకే పాడ్ లో ఉంటాయి.
  * రెండు కంటైనర్లు **ఒకే Network (Same IP & Localhost)** మరియు **Storage Volumes** ని షేర్ చేసుకుంటాయి.
  * నిర్దిష్ట కంటైనర్ లోపలికి లాగిన్ అవ్వడానికి:
    ```bash
    kubectl exec -it multi-container -c nginx -- /bin/sh
    kubectl exec -it multi-container -c almalinux -- bash
    ```

---

## 📌 5. Labels vs Annotations

* [04-labels.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/k8-resources/04-labels.yaml) & [05-annotation.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/k8-resources/05-annotation.yaml)

```yaml
metadata:
  name: demo
  labels:                  # 🏷️ LABELS: Kubernetes సెలెక్టర్లు & ఫిల్టరింగ్ కోసం
    project: roboshop
    component: frontend
    environment: dev
  annotations:             # 📝 ANNOTATIONS: డెవలపర్లు & థర్డ్ పార్టీ టూల్స్ కోసం అదనపు సమాచారం
    jenkinsBuild: "34"
    imageRegistry: "dockerhub"
```
* **ముఖ్యమైన తేడా:** `Service` లోని `selector` కేవలం `labels` ని మాత్రమే గుర్తిస్తుంది, `annotations` ని సెలెక్టర్ గా వాడలేము.

---

## 📌 6. Resource Management: Requests vs Limits

* [06-resource.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/k8-resources/06-resource.yaml)

```yaml
resources:
  requests:          # 🟢 Soft Limit (కనీస రిజర్వేషన్)
    cpu: 100m        # 100 millicores (0.1 Core)
    memory: 128Mi    # 128 Mebibytes
  limits:            # 🔴 Hard Limit (గరిష్ట పరిమితి)
    cpu: 200m        # 200 millicores (0.2 Core)
    memory: 256Mi    # 256 Mebibytes
```
* **ఇంటర్వ్యూ కీ పాయింట్:**
  * **CPU Limit దాటితే:** CPU Throttling జరుగుతుంది (స్లో అవుతుంది కానీ Pod చనిపోదు).
  * **Memory Limit దాటితే:** Pod వెంటనే **OOMKilled (Exit Code 137)** అయ్యి రీస్టార్ట్ అవుతుంది.

---

## 📌 7. Configuration Decoupling: Env, ConfigMaps & Secrets

1. **Direct Env ([07.env.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/k8-resources/07.env.yaml)):** చిన్న టెస్టింగ్ కోసం నేరుగా పాడ్ లోనే `key: value` ఇవ్వడం.
2. **ConfigMap ([08-configmap.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/k8-resources/08-configmap.yaml), [09-pod-configmap.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/k8-resources/09-pod-configmap.yaml)):**
   * కాన్ఫిగరేషన్ డేటాను కోడ్ నుండి వేరు చేయడానికి.
   * `envFrom: - configMapRef: name: nginx-config` ద్వారా అన్ని కీలను ఇంజెక్ట్ చేయవచ్చు.
3. **Secret ([10-secretes.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/k8-resources/10-secretes.yaml), [11-pod-secretes.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/k8-resources/11-pod-secretes.yaml)):**
   * పాస్‌వర్డ్‌లు, API కీలను Base64 లో నిల్వ చేస్తుంది (`echo -n "password" | base64`).
   * పాడ్ లోకి `envFrom: - secretRef: name: nginx-secret` ద్వారా ఇంజెక్ట్ అవుతుంది.

---

## 📌 8. Kubernetes Services & Networking

### 🔹 మూడు రకాల పోర్ట్‌ల పూర్తి వివరణ (Port Mapping)

```
[ Outside Users / Internet ]
            |
            v
     ( Node Public IP : 32141 )  <=== 1. nodePort (30000 - 32767)
            |
            v
      [ Service: nginx-np ]
     ( ClusterIP : Port 80 )     <=== 2. port (Service ClusterIP Port)
            |
            v
         [ Pod ]
     ( Container : Port 80 )     <=== 3. targetPort (Application Port)
```

1. **`targetPort`:** పాడ్ లోపల కంటైనర్ రన్ అవుతున్న పోర్ట్ (ఉదా: NGINX on 80).
2. **`port`:** క్లస్టర్ లోపల సర్వీస్ వినే పోర్ట్ (ClusterIP Port).
3. **`nodePort`:** క్లస్టర్ బయట నుండి వర్కర్ నోడ్ పబ్లిక్ IP ద్వారా బ్రౌజర్‌లో ఓపెన్ చేయడానికి వాడే పోర్ట్.

---

### 🔹 Service Types Comparison

| Service Type | ఫైల్ | ఎక్కడ యాక్సెస్ చేయవచ్చు? | రియల్-వరల్డ్ వాడకం |
| :--- | :--- | :--- | :--- |
| **`ClusterIP`** | [12-service.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/k8-resources/12-service.yaml) | కేవలం క్లస్టర్ లోపల మాత్రమే | Database, Redis, Internal Backend APIs |
| **`NodePort`** | [14-service-np.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/k8-resources/14-service-np.yaml) | `NodeIP:NodePort` ద్వారా బయటి నుండి | Dev/QA టెస్టింగ్, ఇంటర్నల్ టూల్స్ |
| **`LoadBalancer`** | [15-svc-lb.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/k8-resources/15-svc-lb.yaml) | AWS ELB పబ్లిక్ DNS ద్వారా | Production Frontend, Public APIs |

---

## 📌 9. End-to-End Client-to-Service Communication Flow

```
+----------------------------------------------------------------------------------------------------+
|                                         Kubernetes Cluster                                         |
|                                                                                                    |
|   [ 13-service-test.yaml ]                  [ 12-service.yaml ]              [ 04-labels.yaml ]    |
|         (Client)                                 (Proxy)                          (Server)         |
|   +-------------------+                    +------------------+             +------------------+   |
|   | Pod: svc-test     |                    | Service: nginx   |             | Pod: labels      |   |
|   | (AlmaLinux)       | -- curl nginx:80 ->| (ClusterIP: 80)  | ==========> | (NGINX: Port 80) |   |
|   |                   |                    |                  |  Endpoints  | labels:          |   |
|   +-------------------+                    +------------------+   Matched   |   project: robo  |   |
|                                                     |                       |   component: ft  |   |
|                                                     +---------------------> |   environment:dev|   |
|                                                        selector:            +------------------+   |
|                                                          project: robo                             |
|                                                          component: ft                             |
|                                                          environment: dev                          |
+----------------------------------------------------------------------------------------------------+
```

### `curl nginx` కొట్టినప్పుడు స్టెప్స్:
1. `svc-test` పాడ్ CoreDNS ని అడిగి `nginx` సర్వీస్ యొక్క **ClusterIP** ని తెలుసుకుంటుంది.
2. సర్వీస్ లోని `selector` మ్యాచ్ అయిన Pods యొక్క IP లను **Endpoints (`kubectl get ep nginx`)** లో ఉంచుతుంది.
3. సర్వీస్ ఆ రిక్వెస్ట్‌ను NGINX పాడ్ కి ఫార్వర్డ్ చేసి వెబ్‌పేజీ అవుట్‌పుట్‌ను తిరిగి ఇస్తుంది.

---

## 📌 10. ReplicaSets (Self-Healing, Scaling & HA)

* **ReplicaSet ([16-replicaset.yaml](file:///Users/sriramcharankolla/Desktop/DevOps/k8-resources/16-replicaset.yaml)):** ఎల్లప్పుడూ మనం కోరుకున్న సంఖ్యలో ఒకే విధమైన (Identical) Pods రన్ అయ్యేలా చూస్తుంది.

### 🔹 3 ప్రధాన ప్రయోజనాలు (Core Capabilities):
1. **High Availability (HA):** ఒకటి కంటే ఎక్కువ Pods రన్ అవ్వడం వల్ల ఒక Pod ఫెయిల్ అయినా అప్లికేషన్ ఆగదు.
2. **Self-Healing (స్వీయ స్వస్థత):** ఏదైనా కారణం వల్ల ఒక Pod క్రాష్ అయినా లేదా డిలీట్ అయినా, ReplicaSet వెంటనే కొత్త Pod ని ఆటోమేటిక్‌గా లాంచ్ చేస్తుంది.
3. **Scaling:** ట్రాఫిక్ పెరిగినప్పుడు `replicas` కౌంట్‌ను పెంచి లోడ్‌ను సమానంగా పంచుకోవచ్చు:
   ```bash
   kubectl scale rs nginx --replicas=5
   ```

### 🔹 ReplicaSet లో ముఖ్యమైన భాగాలు:
* `replicas: 1` $\rightarrow$ ఎన్ని Pods ఎప్పుడూ రన్ అవ్వాలో నిర్ణయిస్తుంది.
* `selector.matchLabels` $\rightarrow$ ReplicaSet ఏ Pods ని తన కంట్రోల్ లో ఉంచుకోవాలో చెబుతుంది (ఇది `template.metadata.labels` తో ఖచ్చితంగా మ్యాచ్ అవ్వాలి).
* `template` $\rightarrow$ కొత్త Pods క్రియేట్ చేయడానికి ఉపయోగించే బ్లూప్రింట్ (Pod Definition).

### 🔹 ఇంటర్వ్యూ ప్రశ్న: ReplicaSet vs Deployment తేడా ఏమిటి?
> **Answer:** ReplicaSet కేవలం Pod సంఖ్యను (Replicas) స్థిరంగా ఉంచుతుంది, కానీ అప్లికేషన్ వెర్షన్ మారినప్పుడు **Rolling Updates మరియు Zero-Downtime Rollbacks** చేయలేదు. **Deployment** అనేది ReplicaSet పైన ఉండే ఒక హైయర్-లెవెల్ కంట్రోలర్; ఇది ఆటోమేటిక్‌గా పాత ReplicaSet నుండి కొత్త ReplicaSet కి ట్రాఫిక్‌ను డ్రాప్ అవ్వకుండా మార్చగలదు. కాబట్టి ప్రొడక్షన్‌లో ఎప్పుడూ **Deployments** మాత్రమే వాడతారు.

---

## 📌 11. Top Real-World Debugging Errors & Solutions

| ఎర్రర్ మెసేజ్ | కారణం (Root Cause) | పరిష్కారం (Fix) |
| :--- | :--- | :--- |
| `pods "nginx" not found` | పాడ్ వేరే నేమ్‌స్పేస్‌లో ఉంది | `-n roboshop` యాడ్ చేయాలి లేదా కాంటెక్స్ట్ మార్చాలి |
| `curl: (7) Connection refused` | `selector` మరియు `labels` స్పెల్లింగ్ మ్యాచ్ అవ్వలేదు (No endpoints) | `env: dev` మరియు `environment: dev` సరిగ్గా మ్యాచ్ చేయాలి |
| `unknown field "sepc"` / `"secreteRef"` | YAML లో స్పెల్లింగ్ మిస్టేక్ (Typo) | `spec`, `secretRef` గా మార్చాలి |
| `unknown field "limits"` | Indentation లోపం | `requests` మరియు `limits` లను `resources:` కింద స్పేస్‌లతో రాయాలి |
| `unknown field "spec.selector.template"` | `template` ని `selector` లోపల రాయడం | `template` ని `selector` తో సమానమైన లెవెల్‌లో రాయాలి |

---

## 📌 12. Must-Know Kubernetes Interview Q&A

### Q1: Pod మరియు Container మధ్య తేడా ఏమిటి?
> **Answer:** Container అనేది కేవలం ఒక సింగిల్ ప్రాసెస్/యాప్ రన్ అయ్యే ఐసోలేటెడ్ ఎన్విరాన్‌మెంట్. Pod అనేది Kubernetes యొక్క అతి చిన్న మేనేజ్‌మెంట్ యూనిట్. ఒక Pod లోపల ఒకటి లేదా అంతకంటే ఎక్కువ కంటైనర్లు (Sidecar containers) ఒకే IP మరియు స్టోరేజ్‌ను పంచుకుంటూ కలిసి రన్ అవుతాయి.

### Q2: Kubernetes లో Pods చనిపోతే IP మారిపోతుంది కదా, మైక్రోసర్వీసెస్ ఎలా స్థిరంగా మాట్లాడుకుంటాయి?
> **Answer:** Kubernetes **Service** ద్వారా. Service ఒక స్థిరమైన వర్చువల్ IP (ClusterIP) మరియు స్థిరమైన DNS పేరును అందిస్తుంది. బ్యాకెండ్‌లో Pods ఎన్నిసార్లు చనిపోయి కొత్త IP తో వచ్చినా, Service తన `selector` ద్వారా ఆటోమేటిక్‌గా Endpoints ని అప్‌డేట్ చేసుకుంటుంది.

### Q3: ConfigMap లో డేటా మారితే Pod లో ఆటోమేటిక్‌గా మారుతుందా?
> **Answer:** మనం `envFrom` (Environment variables) ద్వారా ఇంజెక్ట్ చేస్తే Pod రీస్టార్ట్ అయ్యే వరకు మారదు. కానీ ConfigMap ను **Volume Mount** రూపంలో మౌంట్ చేస్తే, ఫైల్ కంటెంట్ ఆటోమేటిక్‌గా అప్‌డేట్ అవుతుంది.

### Q4: NodePort పరిధి (Port Range) ఎంత?
> **Answer:** డిఫాల్ట్‌గా `30000` నుండి `32767` వరకు.

### Q5: ReplicaSet ఉన్నప్పుడు మాన్యువల్‌గా ఒక Pod ని డిలీట్ చేస్తే ఏం జరుగుతుంది?
> **Answer:** ReplicaSet Desired State (`replicas: N`) ని నిరంతరం గమనిస్తూ ఉంటుంది. మనం ఒక Pod ని డిలీట్ చేయగానే, ప్రస్తుత Pods సంఖ్య తగ్గడం చూసి వెంటనే క్షణాల్లో మరొక కొత్త Pod ని లాంచ్ చేసి బ్యాలెన్స్ చేస్తుంది (Self-Healing).
