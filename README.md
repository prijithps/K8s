# K8s
K8s interview qns and ans
---

1. **Q:** Your pod is stuck in `CrashLoopBackOff`. How do you debug and fix it?
   **A:**

* Check pod status and events:

  ```bash
  kubectl describe pod <pod-name> -n <ns>
  ```
* Inspect logs of the crashing container(s):

  ```bash
  kubectl logs <pod-name> -c <container-name> -n <ns> --previous
  ```
* If logs are not enough, start an interactive debug container on the same node or copy files:

  ```bash
  kubectl exec -it <pod-name> -c <container-name> -n <ns> -- /bin/sh
  # or
  kubectl debug -it <pod-name> -n <ns> --image=busybox --share-processes
  ```
* Common causes & fixes:

  * **App crash / exception** → fix application bug or configuration (env, args, secrets).
  * **Missing env or secret** → ensure Secret exists and is mounted; check RBAC.
  * **Bad startup command / args** → correct `command`/`args` in the pod spec.
  * **Crash due to readiness/liveness probe misconfiguration** → relax probe thresholds while debugging.
  * **Image pull failures** → check `imagePullPolicy`, image name, registry auth.
  * **Insufficient resources (OOMKilled)** → inspect `kubectl describe` / `kubectl logs` and increase limits/requests.
* Once fixed, update Deployment/DaemonSet/Pod spec and either let controller recreate or delete pod to restart:

  ```bash
  kubectl delete pod <pod-name> -n <ns>
  ```
* Add better telemetry and health checks to prevent recurrence.

---

2. **Q:** How do you perform zero-downtime deployments in Kubernetes?
   **A:**

* Use controllers (Deployment) with rolling updates:

  * Set `strategy.type: RollingUpdate` and tune `maxUnavailable: 0` and `maxSurge` (e.g. `maxSurge: 1`) to avoid losing capacity.
* Ensure **readiness probes** are accurate so only ready pods receive traffic.
* Use **readiness gating** or preStop hooks to drain connections gracefully.
* Use **PodDisruptionBudgets (PDBs)** to maintain minimum available replicas.
* For risk mitigation: Canary or Blue/Green:

  * **Canary**: deploy small percentage, monitor, then promote.
  * **Blue/Green**: deploy new version to new service, switch traffic (Ingress/Service) atomically.
* Automate health checks and metrics gating (e.g., Prometheus alert + CI/CD gates) to rollback on anomalies.
* Use graceful shutdown in app (listen for SIGTERM, finish inflight requests, then exit).

---

3. **Q:** Your service is not accessible externally — where do you start troubleshooting?
   **A:**

* Verify Service object:

  ```bash
  kubectl get svc <svc-name> -o yaml -n <ns>
  ```

  * Check `type` (ClusterIP/NodePort/LoadBalancer) and `externalIPs`/`loadBalancer` status.
* Verify Endpoints/Endpointslices:

  ```bash
  kubectl get endpoints <svc-name> -n <ns>
  ```
* Confirm pod labels match Service selector and pods are `Ready`:

  ```bash
  kubectl get pods -l <selector> -n <ns> -o wide
  ```
* Test connectivity inside cluster:

  * From another pod: `curl http://<svc-name>.<ns>.svc.cluster.local:<port>`
* If using `LoadBalancer`, check cloud LB health and firewall/security groups (ports open).
* If `Ingress` used: check Ingress resource, Ingress controller logs, TLS config, DNS A/CNAME records, and external DNS.
* Check kube-proxy, CNI plugin, and node firewall/iptables rules.
* Check DNS resolution: `nslookup` / `dig` inside a pod.
* Review controller logs (Ingress controller, controller-manager) for errors.

---

4. **Q:** Explain how you'd handle a failed rollout during a deployment.
   **A:**

* Quickly assess rollout status:

  ```bash
  kubectl rollout status deployment/<deploy-name> -n <ns>
  kubectl describe deployment <deploy-name> -n <ns>
  kubectl get events -n <ns>
  ```
* Inspect failing pods (`kubectl logs`, `kubectl describe pod`).
* If immediate rollback needed:

  ```bash
  kubectl rollout undo deployment/<deploy-name> -n <ns>
  ```
* For deeper investigation (non-urgent):

  * Pause rollout (`kubectl rollout pause`) to prevent further changes.
  * Fix image/config/manifest; create a new revision and resume.
* If you use Canary/Automation: stop promotion and investigate telemetry/alerts.
* Post-mortem: identify root cause, update tests/healthchecks, and add CI/CD safety gates.

---

5. **Q:** How would you optimize resource requests and limits in a production cluster?
   **A:**

* Measure real usage with metrics-server, Prometheus, or monitoring stack over representative traffic.
* Choose requests to reflect **steady-state (e.g., 50th percentile)** and limits for spikes (e.g., 95th/99th percentile). Use historical data.
* Use Horizontal Pod Autoscaler (HPA) to react to load; use Vertical Pod Autoscaler (VPA) in recommendation mode or for non-critical workloads.
* Apply LimitRanges and Namespace quotas to prevent noisy neighbors.
* Use QoS classes:

  * `Guaranteed` (requests == limits) for critical pods,
  * `Burstable` for normal pods,
  * `BestEffort` for low-priority pods.
* Start conservative, iterate: deploy changes in staging, run load tests, then apply to prod gradually.
* Use resource requests when scheduling matters — requests affect bin-packing; limits protect against runaway processes.
* Continuously review and automate recommendations (Prometheus + K8s VPA or custom scripts).

---

6. **Q:** How do you secure secrets in Kubernetes?
   **A:**

* Avoid plaintext secrets in manifests or environment variables stored in Git.
* Use Kubernetes `Secret` objects but:

  * **Enable encryption at rest** for Secrets in etcd (KMS provider).
  * Restrict RBAC: grant access only to service accounts that need them.
  * Use `audit` to log secret access attempts.
* Prefer external secret managers for production: HashiCorp Vault, AWS Secrets Manager, Azure Key Vault, GCP Secret Manager.

  * Use Vault Agent Injector, ExternalSecrets Operator, or CSI Secrets Store to mount secrets as files at runtime.
* Use `sealed-secrets` or `sops` for storing encrypted secrets in Git safely.
* Mount secrets as files rather than env vars if logs might leak env vars.
* Rotate secrets regularly and implement automated rotation where possible.
* Run regular scans for accidental secret exposure (e.g., Git scanning).

---

7. **Q:** A node went down suddenly — what happens to the pods running on it?
   **A:**

* Control plane detects node NotReady after node controller timeout (default ~40s–5m depending on config). Pod status becomes `Unknown`/`Terminating` in `kubectl`.
* For pods managed by controllers (Deployment/ReplicaSet/DaemonSet/StatefulSet):

  * Controller will create replacement pods on other healthy nodes once the node is considered `NotReady`/timeout exceeded.
* For pods with **PersistentVolumes** (PVCs):

  * PV behavior depends on storage class and access mode:

    * `ReadWriteOnce`: volume will be detached and reattached on another node (may require that the original node is gone and CSI supports detach).
    * StatefulSet may preserve identity and use PodManagementPolicy to control ordering.
* If node recovers before pod recreation, you can have duplicate pods issue — controllers avoid duplicates by ensuring desired replica count.
* Check `kubectl get pods -o wide` and `kubectl describe node <node>` and the events to confirm and troubleshoot.
* For faster recovery, ensure proper **Cluster Autoscaler** and node pools for capacity and use `PodDisruptionBudgets` to limit simultaneous disruptions.

---

8. **Q:** How do you handle database credentials rotation in Kubernetes?
   **A:**

* Store DB creds in an external secret manager (Vault, AWS Secrets Manager) or Kubernetes Secret with automated rotation integration.
* Rotation pattern:

  1. **Rotate secret in secret store** (generate new password).
  2. **Update Kubernetes Secret** (if using external sync or ExternalSecrets operator this can be automated).
  3. **Trigger rolling restart** of consumers so they pick up new secret:

     * Annotate deployment to force restart:

       ```bash
       kubectl patch deployment <name> -n <ns> -p \
         "{\"spec\": {\"template\": {\"metadata\": {\"annotations\": {\"secret-rotation\":\"$(date +%s)\"}}}}}"
       ```
     * Or use the Secrets Store CSI driver with refresh and sidecar or Vault Agent injector to automatically reload credentials without restarts (if the app supports hot reload).
  4. **Perform DB-side switchover** (e.g., allow both old and new creds for a short overlap or use DB user with limited session tokens).
* Test rotation in staging: verify no dropped connections or graceful reconnect logic.
* Automate and log rotation events; have rollback plan if consumers cannot reconnect.

---

9. **Q:** What's your strategy for backup and restore in a cluster?
   **A:**

* Split backups into two categories:

  1. **Cluster & control plane (etcd)**:

     * Regular etcd snapshots (automated), encrypted, stored off-cluster (S3/Blob).
     * Keep snapshot retention and test restores.
  2. **Application data (stateful data, PVCs, DBs)**:

     * Use application-native backups for databases (logical dumps, PITR for DBs).
     * Use volume snapshots (CSI snapshots) for PVs, coordinated with application quiescing if needed.
* Use tooling like **Velero** for resource manifests + PV snapshots (ensure CSI snapshot support).
* Back up cluster manifests (Deployments, ConfigMaps, CRDs) in GitOps (git) and export cluster/statefulset definitions.
* Automated schedule with offsite replication, encryption, and retention policies.
* Regularly run **restore drills** and document Recovery Time Objective (RTO) and Recovery Point Objective (RPO).
* Maintain runbook: step-by-step restore procedure, credential access, and contact list.

---

10. **Q:** How do you implement auto-scaling when traffic fluctuates heavily?
    **A:**

* Use **Horizontal Pod Autoscaler (HPA)** for pod-level scaling:

  * For CPU/memory: HPA + metrics-server.
  * For custom metrics (requests/sec, queue length): use Prometheus Adapter or custom metrics API.
  * Configure sensible min/max replicas and target utilization.
* Combine with **Cluster Autoscaler** to scale nodes up/down as pod scheduling needs change.
* Use **Vertical Pod Autoscaler (VPA)** for adjusting resource requests over time (use in recommendation mode or for non-critical workloads).
* For latency-sensitive services, implement:

  * **Buffering / queueing** (e.g., message queues) to smooth spikes.
  * Pre-warming or maintaining a small buffer of ready replicas.
* Configure HPA stabilization windows, cooldowns, and scale-up policies to avoid flapping. Use predictive autoscaling if available for frequent predictable patterns.
* Test autoscaling with load tests and monitor SLOs; tune thresholds to balance cost vs performance.

---

