
# 🧠 Kubernetes Class 4 Notes

📅 **Date**: 13 July 2025  
📌 **Topics**: Deployment Continuation, DaemonSets, NodeSelectors, Resource Limits, Taints & Tolerations

---

## 📘 CKA Tip

In the **CKA exam**, you are allowed to access Kubernetes official documentation, but **only one tab** is allowed.

---

## 🚀 Deployment Continuation

### 1. Exposing Deployment

```bash
kubectl expose deployment nginx-app --port=80 --type=NodePort
```

### 2. Editing Deployment

```bash
kubectl edit deploy nginx-app
```

- Under the template `image:` section, change `nginx` to `caddy` or any new image.
- This triggers a rolling update and lets us observe the rollout process.

### 3. Rollout Status

```bash
kubectl rollout status deployment nginx-app
```

### 4. Rollout Pause and Resume

```bash
kubectl rollout pause deployment nginx-app
kubectl rollout resume deployment nginx-app
```

- Use pause when performing major changes and validate replicas.
- Resume only after validation.
- To **undo**, rollout must be fully completed.

### 5. `minReadySeconds` Parameter

- Ensures pods are available before continuing rollout.
- Prevents incomplete rollouts due to unready pods.

---

## 📦 DaemonSets

### Overview

1. Ensures one pod replica runs on **each node**.
2. Commonly used for:
   - Logging agents
   - Monitoring agents (Prometheus, etc.)
   - Networking plugins (Calico, kube-proxy)
3. Unlike ReplicaSet, which may place all pods on one node, DaemonSets distribute pods to all nodes.
4. Use DaemonSets when workloads **must run on all nodes**.

### Check Resource

```bash
kubectl api-resources | grep -iE "KIND|Daemon"
```

### Sample DaemonSet Manifest

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

---

## 📍 NodeSelector

### Purpose:

Assign pods to specific nodes using node labels.

### Use Cases:

- Run workloads on SSD/GPU nodes
- Schedule to Linux-only worker nodes
- Control pod placement manually

### Example DaemonSet with NodeSelector

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-fast-storage
  labels:
    app: nginx
    ssd: "true"
spec:
  selector:
    matchLabels:
      ssd: "true"
  template:
    metadata:
      labels:
        app: nginx
        ssd: "true"
    spec:
      containers:
      - name: nginx
        image: nginx
      nodeSelector:
        env: "true"
```

### Commands

```bash
kubectl create -f nginx-daemonset.yaml
kubectl label node worker.example.net env=true
kubectl label node worker.example.net env-
kubectl delete -f nginx-daemonset.yaml
```

---

## 📊 Resource Requests and Limits

### Definitions

- **Limits**: Maximum allowed resources (Hard cap)
- **Requests**: Minimum guaranteed resources (Soft reservation)

### YAML Example

```yaml
resources:
  limits:
    memory: 200Mi
  requests:
    cpu: 100m
    memory: 200Mi
```

### Notes

- Pods without limits can overconsume resources.
- Pods with unmet requests will remain in `Pending`.
- Enforced using **cgroups** in container runtimes.

---

## ⚠️ Taints and Tolerations

### Concepts

- **Taints**: Applied on nodes to repel pods.
- **Tolerations**: Added to pods to allow scheduling on tainted nodes.

### Effects of Taints

1. `NoSchedule`: Block new pods
2. `PreferNoSchedule`: Avoid scheduling unless necessary
3. `NoExecute`: Evict non-tolerating pods

### Master Node Taint Example

```bash
kubectl describe node master.example.net
# Shows: Taints: node-role.kubernetes.io/controlplane:NoSchedule
```

### Tainting and Untainting Nodes

```bash
kubectl taint node worker.example.net gpu=true:NoSchedule
kubectl taint node worker.example.net gpu-
```

### Sample Pod with Toleration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx-pod
    image: nginx
  tolerations:
  - key: gpu
    operator: "Exists"
    effect: NoSchedule
```

---
