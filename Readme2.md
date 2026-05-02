
---

# 🔹 Basic Kubernetes Interview Questions

### 1. What is Kubernetes?

**Answer:**
Kubernetes is an open-source container orchestration platform used to automate deployment, scaling, and management of containerized applications.

---

### 2. What are the main components of Kubernetes architecture?

**Answer:**

**Master (Control Plane):**

* API Server
* Scheduler
* Controller Manager
* etcd (key-value store)

**Worker Node:**

* Kubelet
* Kube-proxy
* Container runtime (Docker/containerd)

---

### 3. What is a Pod?

**Answer:**
A **Pod** is the smallest deployable unit in Kubernetes. It can contain one or more containers that share:

* Network (same IP)
* Storage (volumes)

---

### 4. Difference between Deployment and StatefulSet?

**Answer:**

| Feature      | Deployment     | StatefulSet        |
| ------------ | -------------- | ------------------ |
| Use case     | Stateless apps | Stateful apps      |
| Pod identity | Random         | Stable             |
| Storage      | Shared         | Persistent per pod |
| Scaling      | Easy           | Ordered            |

---

### 5. What is a Service in Kubernetes?

**Answer:**
A **Service** exposes a set of Pods and provides stable networking.

Types:

* ClusterIP
* NodePort
* LoadBalancer
* ExternalName

---

# 🔹 Intermediate Kubernetes Questions

### 6. What is a Namespace?

**Answer:**
A **Namespace** is a logical isolation in a cluster used to organize resources.

Example:

```bash
kubectl get pods -n dev
```

---

### 7. What is ConfigMap and Secret?

**Answer:**

| Feature   | ConfigMap   | Secret            |
| --------- | ----------- | ----------------- |
| Data type | Plain text  | Sensitive data    |
| Encoding  | Not encoded | Base64 encoded    |
| Use case  | Config      | Passwords, tokens |

---

### 8. What is Ingress?

**Answer:**
Ingress manages **external HTTP/HTTPS access** to services inside the cluster.

---

### 9. What is a DaemonSet?

**Answer:**
Ensures a pod runs on **every node**.

Use cases:

* Log collectors
* Monitoring agents

---

### 10. What is Horizontal Pod Autoscaler (HPA)?

**Answer:**
Automatically scales pods based on:

* CPU
* Memory
* Custom metrics

---

# 🔹 Advanced Kubernetes Questions

### 11. How does Kubernetes networking work?

**Answer:**

* Each Pod gets a unique IP
* Pods can communicate directly
* Uses CNI plugins (Calico, Flannel)

---

### 12. What is etcd?

**Answer:**
**etcd** is a distributed key-value store used to store cluster state.

---

### 13. What are Taints and Tolerations?

**Answer:**

* **Taint**: Restrict nodes
* **Toleration**: Allow pods to be scheduled

Example:

```bash
kubectl taint nodes node1 key=value:NoSchedule
```

---

### 14. What is a Persistent Volume (PV) and Persistent Volume Claim (PVC)?

**Answer:**

* **PV** → Actual storage
* **PVC** → Request for storage

---

### 15. What is RBAC?

**Answer:**
Role-Based Access Control manages permissions.

Components:

* Role / ClusterRole
* RoleBinding / ClusterRoleBinding

---

# 🔹 Scenario-Based Questions (Important for Interviews)

### 16. Pod is not starting. How do you debug?

**Answer:**

```bash
kubectl describe pod <pod>
kubectl logs <pod>
kubectl get events
```

Check:

* Image issues
* CrashLoopBackOff
* Resource limits

---

### 17. How do you perform rolling updates?

**Answer:**

```bash
kubectl set image deployment app container=image:v2
```

Kubernetes ensures:

* Zero downtime
* Gradual rollout

---

### 18. How do you scale a deployment?

**Answer:**

```bash
kubectl scale deployment app --replicas=5
```

---

### 19. How do you expose an application externally?

**Answer:**

* NodePort
* LoadBalancer
* Ingress (preferred)

---

### 20. How do you monitor Kubernetes?

**Answer:**

* Prometheus
* Grafana
* ELK Stack

---

# 🔹 DevOps/SRE Level Deep Questions

### 21. How does Kubernetes handle self-healing?

**Answer:**

* Restarts failed containers
* Replaces unhealthy pods
* Reschedules pods on failure

---

### 22. What is the difference between Docker Swarm and Kubernetes?

**Answer:**

| Feature    | Docker Swarm | Kubernetes |
| ---------- | ------------ | ---------- |
| Complexity | Simple       | Complex    |
| Scaling    | Basic        | Advanced   |
| Ecosystem  | Limited      | Huge       |

---

### 23. What is a Helm Chart?

**Answer:**
Helm is used to manage Kubernetes applications via charts.

---

### 24. What is a Sidecar Container?

**Answer:**
A **sidecar** runs alongside the main container for:

* Logging
* Monitoring
* Proxy

---

### 25. How do you secure Kubernetes?

**Answer:**

* RBAC
* Network Policies
* TLS
* Secrets management
* Pod Security Standards

---

