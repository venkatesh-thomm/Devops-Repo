
# Kubernetes Complete Internal & External Access Guide

This README merges all explanations discussed in this chat.

---

## 🚀 Kubernetes Ports & Rules

### containerPort
The port exposed internally inside container.

### targetPort
Service sends traffic to this port inside container.

### Must Match Example
containerPort = targetPort = 3000
containerPort must equal targetPort → YES
---

## ❌ INVALID PORT EXAMPLE

targetPort: 300003 ← Not allowed

Ports allowed only: 1 – 65535

---

# 🌐 INTERNAL ACCESS (ClusterIP)

- Default service
- Used for internal microservice communication
- Pods access via DNS

### Access:
http://<service-name>:<port>

---

## YAML EXAMPLE

apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector:
    app: backend
  ports:
  - port: 3000
    targetPort: 3000

---

# 🌍 EXTERNAL ACCESS (NodePort)

- Allows browser access

### Access format:
http://<NodeIP>:<NodePort>

---

## YAML EXAMPLE

apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
  - port: 3000
    targetPort: 3000
    nodePort: 31000

---

# 🌐 LOADBALANCER ACCESS

- Cloud solution using LB

---

# 🔥 TROUBLESHOOTING NODEPORT

## 1️⃣ Check nodes
kubectl get nodes -o wide

## 2️⃣ Check services
kubectl get svc -o wide

## 3️⃣ Check endpoints
kubectl get endpoints <svc>

## 4️⃣ Test inside cluster
kubectl run tmp --rm -it --image=busybox sh
wget <service>:3000

## 5️⃣ Ensure app listens 0.0.0.0
kubectl exec -it <pod> -- netstat -tulpn

---

# ❗ WHY NODEPORT FAILS OUTSIDE?

- No public node IP
- Cloud firewall blocked
- Wrong NodeIP tested
- VM firewall blocked
- Service not reachable outside

---

# ✔️ STEP BY STEP COMMANDS

kubectl get pods -o wide
kubectl get svc -o wide
kubectl apply -f deployment.yaml
kubectl apply -f service-clusterip.yaml
kubectl run tmp --rm -it --image=busybox sh
wget <service-name>:<port>
kubectl apply -f service-nodeport.yaml

---

# 🧪 VALIDATION CHECKLIST

- Pod running
- Pod Ready:true
- App logs show healthy
- Namespaces match
- Labels match
- Endpoints not empty
- Port forwarding correct

---

# ⭐ LABELS MUST MATCH

deployment labels:
app: frontend

service selector:
app: frontend

---

# 📌 FINAL WORKFLOW

ClusterIP → internal communication  
NodePort → external node access  
LoadBalancer → cloud public access  

---

# ✅ Step-1: Check Pod is actually listening on the port

```bash
kubectl exec -it <pod-name> -- netstat -tulpn
```
> Check app is listening on 0.0.0.0:3000 (not only localhost).

> If app listens only on 127.0.0.1:3000, service cannot reach it.


# ✅ Step-2: Check Pod IP is part of endpoints 

```bash
kubectl get endpoints <service-name> -o wide
```

> If endpoints are empty → service cannot route to pod.

> Common reasons:  labels mismatch and selectors mismatch

# ✅ Step-3: Check Service selector labels

```bash
Deployment:

labels:
  app: frontend


Service:

selector:
  app: frontend


If they differ even by one letter → service will break.
```

# ✅ Step-4: Check Service type

```bash
If you want access from outside:

NodePort example:

port: 3000 → cluster

nodePort: 31000 → external

You must curl nodeIP:31000
```

# ✅ Step-5: Try curling inside the cluster
```bash
kubectl run tmp --rm -it --image=busybox sh
wget <service-name>:3000

If internal works but external doesn’t → service type issue.

```

# ✅ Step-6: Check container logs
```bash
kubectl logs <pod-name>


If app fails to start → no response.


| Issue                         | Why service fails          |
| ----------------------------- | -------------------------- |
| Wrong selector                | No endpoints               |
| App listens only on localhost | Service cannot connect     |
| Pod not ready                 | Removed from LB            |
| Wrong namespace               | Service cannot find pod    |
| port mismatch                 | Traffic drops              |
| not using NodePort            | Can’t access from external |

---

| Purpose                                         | How we access it                | Service type used        |
| ----------------------------------------------- | ------------------------------- | ------------------------ |
| Inside the cluster                              | Pod → Pod OR Pod → Service      | ClusterIP                |
| Outside the cluster (browser, mobile, internet) | NodeIP / LoadBalancer / Ingress | NodePort or LoadBalancer |

```
---


# 🌐 INTERNAL ACCESS (Service = ClusterIP)
```bash
This is the default type.
Used when only pods within cluster need to communicate.
You CANNOT reach it from laptop/host browser.

Example use case:

frontend talks to backend
OR
backend talks to database

Example Deployment:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: my-backend
        ports:
        - containerPort: 3000

Example SERVICE — internal:
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector:
    app: backend
  ports:
  - port: 3000
    targetPort: 3000

🔹 Result:

Other pods can access using:

http://backend:3000


But you CANNOT access it externally.
```

---


# 🌍 EXTERNAL ACCESS (TYPE = NodePort)

```bash 
Used for exposing app outside the cluster (browser / postman).

How it works:

NodePort opens a port (30000–32767) on all worker nodes.

Example:
Deployment:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: my-frontend
        ports:
        - containerPort: 3000

Service (external using NodePort):
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
  - port: 3000
    targetPort: 3000
    nodePort: 31000

How to Access:
http://<NodeIP>:31000


NodeIP = worker node public/private IP.

That gives external access.
```
----


# 🌐 EXTERNAL ACCESS (TYPE = LoadBalancer)
```bash
Best in cloud environments (AWS EKS, GKE, AKS).

Service interacts with a cloud Load Balancer.

Example Service:
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 3000

How to access:
http://<Service EXTERNAL-IP>
```

# ⭐ VISUAL COMPARISON
```bash
Internal (ClusterIP)
Pod → Service → Pod


Cannot access from browser.

NodePort
Browser/Host → NodeIP:NodePort → Service → Pod

LoadBalancer
Browser/Internet → Load Balancer → Service → Pod
```
---

# WHEN TO USE WHAT?
```bash 
| Scenario              | Best Option              |
| --------------------- | ------------------------ |
| frontend web app      | LoadBalancer or NodePort |
| database              | ClusterIP                |
| backend microservices | ClusterIP                |
| internal APIs         | ClusterIP                |
| dev/test environments | NodePort                 |
| production cloud apps | LoadBalancer + Ingress   |

```
---
```bash
Deployment
 ├─ apiVersion: apps/v1
 ├─ kind: Deployment
 ├─ metadata
 │    ├─ name
 │    ├─ namespace
 │    └─ labels
 └─ spec
      ├─ replicas: <number of pods>
      ├─ selector
      │     └─ matchLabels
      └─ template
           ├─ metadata
           │     └─ labels
           └─ spec
                 └─ containers
                       ├─ name
                       ├─ image
                       ├─ ports
                       │     └─ containerPort
                       ├─ resources
                       │     ├─ requests (cpu/memory)
                       │     └─ limits (cpu/memory)
                       └─ probes
                             ├─ livenessProbe
                             └─ readinessProbe

---
HPA
 ├─ apiVersion: autoscaling/v2
 ├─ kind: HorizontalPodAutoscaler
 ├─ metadata
 │    └─ name
 └─ spec
      ├─ scaleTargetRef (Deployment to scale)
      │     ├─ apiVersion: apps/v1
      │     ├─ kind: Deployment
      │     └─ name: <deployment-name>
      ├─ minReplicas
      ├─ maxReplicas
      └─ metrics
            └─ type: Resource
                  ├─ name: cpu
                  └─ target: averageUtilization (%)
---                  