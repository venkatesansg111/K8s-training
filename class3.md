
# ✅ Kubernetes Class 3 Notes

📅 **Date**: 11 July 2025  
🕒 **Time**: 11:15 AM

---

## 1. 🔐 Kubeconfig Authentication with `--kubeconfig`

---

## 2. 🧰 Enhanced Grep for `kubectl` Command-Line Help

```bash
kubectl api-resources | grep -iE "KIND|pod"
kubectl explain pod | head -n 3
```

---

## 3. 🏷️ Labels, Selectors, and Annotations

### a. Equality-Based Selectors

```yaml
env=dev
env!=prod
game=gtavicecity
```

### b. Set-Based Selectors

```yaml
env in (production, qa)
tier notIn (frontend, backend)
```

### 📌 Key Notes:

- **Labels**: Queryable key-value pairs used for identifying/filtering Kubernetes objects.
- **Annotations**: Used to store non-queryable metadata (like notes or references).
- **Selectors**: Used to group objects based on labels (e.g., Services → Pods with `env=dev`).

### 🧪 Commands

```bash
kubectl get pods --show-labels
kubectl get pods -l env=dev
kubectl get deploy --selector tier!=frontend
kubectl get pods -l "env in (dev, prod)"
```

### 🖊️ Add/Remove Labels

```bash
kubectl label pod nginx-pod env=qa
kubectl label pod nginx-pod env-     # Remove label
```

---

## 4. 📄 Pod Manifest with `--dry-run`

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx.yaml
```

> Any pod not maintained by Deployment, ReplicaSet, or Controller is called a **naked pod**.

---

## 5. ⚙️ Controllers

---

### a. ReplicationController (Deprecated)

> Replaced by ReplicaSet. Used to maintain a specific number of replicas.

### Key Spec Components

1. `replicas`
2. `selector`
3. `template`

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    env: dev
    app: frontend
spec:
  replicas: 3
  selector:
    app: frontend
  template:
    metadata:
      name: myapp-pod
      labels:
        app: frontend
        env: dev
    spec:
      containers:
      - name: myapp-pod
        image: nginx
```

### 🧪 RC Commands

```bash
kubectl get rc
kubectl get rc myapp-rc
kubectl describe rc myapp-rc
kubectl edit rc myapp-rc
kubectl scale rc myapp-rc --replicas=4
kubectl delete rc myapp-rc --cascade=orphan  # Leaves naked pods
```

---

### b. ReplicaSet (Preferred)

> Supports multiple label selectors using `matchLabels` and `matchExpressions`.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-rs
  labels:
    env: prod
spec:
  replicas: 3
  selector:
    matchExpressions:
    - key: app
      operator: In
      values:
      - frontend
  template:
    metadata:
      name: myapp-pod
      labels:
        app: frontend
    spec:
      containers:
      - name: nginx-pod
        image: nginx
```

### 🆘 Help

```bash
kubectl explain ReplicaSet.spec.selector
```

---

## 6. 🚀 Deployment

> Manages ReplicaSet + supports rollout strategies (update, rollback, history)

### ✨ Deployment Capabilities

- Auto-creates ReplicaSet
- Rollout status, undo, history
- Strategy types: `RollingUpdate` (default), `Recreate`, `Blue/Green`, `Canary`

```bash
kubectl api-resources | grep -iE "KIND|deploy"
kubectl create deployment nginx-deploy --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

### Deployment YAML Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
  labels:
    app: nginx-app
    env: dev
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      name: nginx-app
      labels:
        app: nginx-app
        env: dev
    spec:
      containers:
      - name: nginx-app
        image: nginx:1.28-alpine
```

---

### 🔁 RollingUpdate Strategy

- `maxSurge`: Pods added during update (extra capacity)
- `maxUnavailable`: Pods allowed to go down during update
- Supports **zero downtime deployment**

### ⏳ Rollout Management

```bash
kubectl rollout history deployment nginx-app
kubectl apply -f nginx-deployment.yaml --record
kubectl rollout status deployment nginx-app
kubectl rollout undo deployment nginx-app --to-revision=1
```

---
