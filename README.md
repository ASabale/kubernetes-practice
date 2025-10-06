# Kubernetes Learning Guide

Welcome! This guide is meant to give you a structured path for learning Kubernetes (K8s) after you have a local cluster running (for example with Docker Desktop, minikube, or kind). It combines conceptual explanations, hands-on milestones, and suggestions for further exploration.

If you want to dive straight into a practical walkthrough, the repository now includes a [hands-on demo](demo/README.md) with ready-to-apply manifests that exercise Deployments, Services, ConfigMaps, Secrets, Jobs, CronJobs, HPAs, and StatefulSets. You can apply everything with `kubectl apply -k demo/manifests`.

---

## 1. Core Concepts

### 1.1 What is Kubernetes?
Kubernetes is an open-source orchestration platform that automates the deployment, scaling, and management of containerized applications. It abstracts the underlying infrastructure so you can describe your system declaratively (desired state) and let Kubernetes keep it running.

### 1.2 Kubernetes Architecture Overview
- **Control Plane** – Maintains cluster state.
  - `kube-apiserver`: REST API and front door to the cluster.
  - `etcd`: Highly available key-value store for cluster state.
  - `kube-scheduler`: Assigns Pods to Nodes.
  - `kube-controller-manager`: Runs controllers that reconcile desired vs. current state.
  - `cloud-controller-manager`: Integrates with cloud providers (optional for local setups).
- **Worker Nodes** – Run your workloads.
  - `kubelet`: Agent ensuring containers described in Pod specs are running.
  - `kube-proxy`: Handles networking, implementing Services.
  - `container runtime`: Runs containers (containerd, Docker, etc.).

### 1.3 Kubernetes API Primitives
- **Namespace** – Logical partitioning of cluster resources.
- **Pod** – Smallest deployable unit; one or more containers sharing storage/network.
- **ReplicaSet** – Ensures a specified number of Pod replicas are running.
- **Deployment** – Manages ReplicaSets and allows declarative updates.
- **DaemonSet** – Runs a Pod copy on every (or selected) node.
- **StatefulSet** – Manages stateful applications with stable network identities and persistent storage.
- **Job/CronJob** – Run finite or scheduled tasks.
- **Service** – Stable networking abstraction exposing Pods.
- **Ingress** – HTTP(S) routing into cluster.
- **ConfigMap/Secret** – Inject configuration or sensitive data.
- **PersistentVolume / PersistentVolumeClaim** – Abstract storage.

---

## 2. Getting Hands-On

### 2.1 Verify Your Cluster
1. `kubectl version --short`
2. `kubectl cluster-info`
3. `kubectl get nodes`

### 2.2 First Deployment
1. Create a Namespace: `kubectl create namespace demo`
2. Deploy a simple app:
   ```bash
   kubectl create deployment hello --image=nginxdemos/hello -n demo
   kubectl expose deployment hello --type=ClusterIP --port=80 --target-port=80 -n demo
   ```
3. Explore resources: `kubectl get all -n demo`
4. Port-forward to test locally: `kubectl port-forward svc/hello 8080:80 -n demo`

### 2.3 Declarative Manifests
1. Write YAML for the Deployment and Service.
2. Apply it: `kubectl apply -f deployment.yaml`
3. Update image/tag and re-apply to practice rolling updates.

### 2.4 Observability Basics
- `kubectl describe <resource>` for detailed state.
- `kubectl logs <pod>` for application logs.
- Use `kubectl top nodes/pods` with Metrics Server installed.

---

## 3. Understanding Networking

### 3.1 Pod Networking
- Pods get an IP address from the cluster network; they can reach each other directly.
- The CNI (Container Network Interface) plugin (e.g., Flannel, Calico) sets up networking.

### 3.2 Services
- **ClusterIP** – Internal-only stable IP.
- **NodePort** – Exposes service on each node IP at a static port.
- **LoadBalancer** – Provisions external load balancers (depends on environment).
- **Headless Service** – Used for direct Pod access (e.g., stateful apps).

### 3.3 Ingress Controllers
- Provide HTTP/HTTPS routing based on host/path rules.
- Popular controllers: NGINX Ingress Controller, Traefik, Istio ingress gateway.

---

## 4. Configuring Applications

### 4.1 ConfigMaps and Secrets
- Store configuration separate from images.
- Mount as environment variables or files.
- Secrets are base64-encoded; use dedicated secret managers for production.

### 4.2 Environment-Specific Configuration
- Use multiple overlays (e.g., with [Kustomize](https://kustomize.io/) or Helm charts) to manage different environments.

### 4.3 Resource Requests & Limits
- Define CPU/memory requests (minimum guaranteed) and limits (maximum allowed).
- Proper sizing enables effective scheduling and cluster stability.

---

## 5. Storage

### 5.1 Volumes
- **emptyDir** – Ephemeral storage tied to Pod lifecycle.
- **hostPath** – Direct mount from node filesystem (not portable).
- **PersistentVolume** – Cluster-level storage resource.

### 5.2 PersistentVolumeClaims (PVC)
- Pods request storage via PVCs; Kubernetes binds to a matching PV.

### 5.3 StorageClasses & Dynamic Provisioning
- StorageClasses define provisioners and parameters.
- With dynamic provisioning, PVCs automatically create PVs.

### 5.4 Stateful Applications
- Use StatefulSets + PVC templates.
- Consider backup/restore strategies.

---

## 6. Scaling and Updates

### 6.1 Horizontal Pod Autoscaler (HPA)
- Scales Pods based on metrics (CPU, custom metrics).
- `kubectl autoscale deployment hello --min=2 --max=5 --cpu-percent=60`

### 6.2 Vertical Pod Autoscaler (VPA)
- Suggests or automatically adjusts resource requests/limits.

### 6.3 Cluster Autoscaler
- Adds/removes nodes (in cloud environments).

### 6.4 Rolling Updates & Rollbacks
- `kubectl rollout status deployment/hello`
- `kubectl rollout history deployment/hello`
- `kubectl rollout undo deployment/hello`

---

## 7. Security Fundamentals

### 7.1 RBAC (Role-Based Access Control)
- **Role/ClusterRole** – Define permissions.
- **RoleBinding/ClusterRoleBinding** – Attach roles to users/groups/service accounts.

### 7.2 Service Accounts
- Identity for Pods; used for API access.

### 7.3 Network Policies
- Define allowed ingress/egress traffic between Pods.

### 7.4 Pod Security
- Pod Security Standards (Baseline, Restricted) or Admission Controllers.
- Use securityContext: run as non-root, drop capabilities, read-only root filesystem.

### 7.5 Secret Management
- Integrate with external secret stores (e.g., HashiCorp Vault, AWS Secrets Manager).

---

## 8. Observability & Troubleshooting

### 8.1 Logging
- Centralize logs with Fluentd, Fluent Bit, or Logstash.
- Use EFK (Elasticsearch, Fluentd, Kibana) or Loki/Grafana stacks.

### 8.2 Metrics & Monitoring
- Prometheus + Alertmanager for metrics and alerts.
- Grafana for visualization.
- Kubernetes Dashboard (with caution for production).

### 8.3 Tracing
- Distributed tracing with Jaeger or Zipkin; integrate via service mesh or instrumentation.

### 8.4 Debugging Tips
- `kubectl describe` and `kubectl logs`.
- `kubectl exec -it <pod> -- /bin/sh` for interactive debugging (if permitted).
- Use ephemeral containers (`kubectl debug`) for troubleshooting.

---

## 9. Advanced Topics

### 9.1 Helm
- Package manager for Kubernetes; templating & release management.
- Explore `helm create`, `helm install`, `helm upgrade`.

### 9.2 Kustomize
- Overlay-based configuration management built into `kubectl`.

### 9.3 Operators & CRDs
- Extend Kubernetes with Custom Resource Definitions and controllers (Operators).

### 9.4 Service Mesh
- Istio, Linkerd, Consul for advanced traffic management, security, observability.

### 9.5 GitOps
- Manage deployments via Git (e.g., Argo CD, Flux).

### 9.6 Multi-Cluster & Federation
- Manage multiple clusters, global services, failover.

---

## 10. Practice Roadmap

1. **Week 1 – Basics**: Pods, Deployments, Services, Namespaces.
2. **Week 2 – Configuration & Storage**: ConfigMaps, Secrets, PVCs.
3. **Week 3 – Scaling & Updates**: HPA, rollouts, resource tuning.
4. **Week 4 – Security & Observability**: RBAC, Network Policies, monitoring stack.
5. **Week 5 – Advanced**: Helm, Operators, GitOps.
6. **Capstone**: Deploy a multi-tier app with CI/CD pipeline and monitoring.

Track progress by keeping manifests in version control and writing post-mortems for any issues you encounter.

---

## 11. Helpful Tools & Resources

- **Documentation**: [kubernetes.io/docs](https://kubernetes.io/docs/)
- **Kubernetes the Hard Way** (Kelsey Hightower)
- **Playgrounds**: [Katacoda (O'Reilly)](https://www.oreilly.com/online-learning/), [killercoda.com](https://killercoda.com/)
- **Courses**: CNCF Certified Kubernetes Administrator (CKA) curriculum, Kubernetes By Example.
- **Books**: *Kubernetes Up & Running*, *The Kubernetes Book*.
- **CLI Enhancements**: `kubectx`, `kubens`, `k9s`, `stern`.

---

## 12. Next Steps

- Automate manifest management (Helm/Kustomize).
- Experiment with Stateful applications and storage classes.
- Explore CI/CD pipelines that deploy to your cluster.
- Set up monitoring and alerts.
- Contribute to open-source Kubernetes ecosystem projects.

Happy learning! Keep practicing with real workloads and gradually adopt production-grade patterns as your confidence grows.

