# Kubernetes Learning Guide

Welcome! This guide is meant to give you a structured path for learning Kubernetes (K8s) after you have a local cluster running (for example with Docker Desktop, minikube, or kind). It combines conceptual explanations, hands-on milestones, and suggestions for further exploration.

If you want to dive straight into a practical walkthrough, the repository now includes a [hands-on demo](demo/README.md) with ready-to-apply manifests that exercise Deployments, Services, ConfigMaps, Secrets, Jobs, CronJobs, HPAs, and StatefulSets. You can apply everything with `kubectl apply -k demo/manifests`.

---

## 1. Learning Path Overview

### 1.1 Prerequisites & Mindset
- Comfort with Docker or other container runtimes.
- Basic Linux commands, networking fundamentals, and YAML.
- Familiarity with Git for versioning manifest changes.
- Adopt a **declarative mindset**: describe the state you want, then rely on Kubernetes controllers to reconcile it.

### 1.2 Suggested Learning Rhythm
1. Read the summary of a concept.
2. Apply it in your own cluster (minikube/kind or a managed cluster).
3. Observe the resulting resources with `kubectl`.
4. Iterate: tweak the manifest, reapply, and watch Kubernetes converge.
5. Reflect: document key learnings and commands in your own notes or wiki.

### 1.3 Additional Tooling
- Install [`kubectl`](https://kubernetes.io/docs/tasks/tools/), [`kubectx/kubens`](https://github.com/ahmetb/kubectx) for quick context/namespace switching, and [`stern`](https://github.com/stern/stern) for log aggregation.
- Consider [`k9s`](https://k9scli.io/) or Lens for a visual interface.
- Use [`kind`](https://kind.sigs.k8s.io/) or [`minikube`](https://minikube.sigs.k8s.io/) for local multi-node clusters.

---

## 2. Core Concepts

### 2.1 What is Kubernetes?
Kubernetes is an open-source orchestration platform that automates the deployment, scaling, and management of containerized applications. It abstracts the underlying infrastructure so you can describe your system declaratively (desired state) and let Kubernetes keep it running.

### 2.2 Kubernetes Architecture Overview
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

### 2.3 Kubernetes API Primitives
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

### 2.4 Declarative vs. Imperative
- Imperative commands (e.g., `kubectl create deployment ...`) are great for experimentation.
- Declarative workflows use YAML manifests, Git, and pipelines to track changes and enable GitOps.
- Learn both, but practice writing manifests for repeatability and collaboration.

---

## 3. Getting Hands-On

### 3.1 Verify Your Cluster
1. `kubectl version --short`
2. `kubectl cluster-info`
3. `kubectl get nodes`
4. Inspect components: `kubectl get pods -A`
5. Optional: Enable metrics with `minikube addons enable metrics-server` or the equivalent for your environment.

### 3.2 First Deployment (Imperative)
1. Create a Namespace: `kubectl create namespace demo`
2. Deploy a simple app:
   ```bash
   kubectl create deployment hello --image=nginxdemos/hello -n demo
   kubectl expose deployment hello --type=ClusterIP --port=80 --target-port=80 -n demo
   ```
3. Explore resources: `kubectl get all -n demo`
4. Port-forward to test locally: `kubectl port-forward svc/hello 8080:80 -n demo`
5. Scale imperatively: `kubectl scale deployment hello --replicas=3 -n demo`

### 3.3 First Deployment (Declarative)
1. Write YAML for the Deployment and Service.
2. Apply it: `kubectl apply -f deployment.yaml`
3. Update image/tag and re-apply to practice rolling updates.
4. Track rollout status: `kubectl rollout status deployment/hello -n demo`
5. View YAML for a live object: `kubectl get deployment hello -n demo -o yaml`

### 3.4 Observability Basics
- `kubectl describe <resource>` for detailed state.
- `kubectl logs <pod>` for application logs.
- Use `kubectl top nodes/pods` with Metrics Server installed.
- Tail logs across replicas with `stern hello -n demo`.
- Collect events: `kubectl get events -n demo --sort-by=.metadata.creationTimestamp`.

### 3.5 Clean Up
- Delete namespace to remove all resources: `kubectl delete namespace demo`.
- Practice idempotency: reapply manifests and confirm Kubernetes reconciles state without error.

---

## 4. Understanding Networking

### 4.1 Pod Networking
- Pods get an IP address from the cluster network; they can reach each other directly.
- The CNI (Container Network Interface) plugin (e.g., Flannel, Calico) sets up networking.
- Understand the pod-to-pod communication model: no NAT within the cluster CIDR.
- Troubleshoot connectivity with ephemeral busybox pods: `kubectl run -it tmp --rm --image=busybox --restart=Never -- sh`.

### 4.2 Services
- **ClusterIP** – Internal-only stable IP.
- **NodePort** – Exposes service on each node IP at a static port.
- **LoadBalancer** – Provisions external load balancers (depends on environment).
- **Headless Service** – Used for direct Pod access (e.g., stateful apps).
- Explore endpoints: `kubectl get endpointslice -n demo`.

### 4.3 Ingress Controllers
- Provide HTTP/HTTPS routing based on host/path rules.
- Popular controllers: NGINX Ingress Controller, Traefik, Istio ingress gateway.
- Practice: deploy an ingress controller (e.g., `minikube addons enable ingress`) and expose your app with host rules.
- Understand TLS termination and certificate management (cert-manager, ACME).

### 4.4 Service Mesh (Optional Advanced Topic)
- Service meshes (Istio, Linkerd, Consul) add traffic management, security, and observability layers.
- Learn concepts such as sidecar proxies, mTLS, and traffic splitting.

---

## 5. Configuring Applications

### 5.1 ConfigMaps and Secrets
- Store configuration separate from images.
- Mount as environment variables or files.
- Secrets are base64-encoded; use dedicated secret managers for production.
- Use `kubectl create configmap ... --from-literal` for quick experiments.
- Practice reloading: change ConfigMap data and restart pods (or use projected volumes with reloaders).

### 5.2 Environment-Specific Configuration
- Use multiple overlays (e.g., with [Kustomize](https://kustomize.io/) or Helm charts) to manage different environments.
- Store manifests in Git; use branches or overlays for dev/staging/prod.
- Learn Helm basics: charts, values, templating, release lifecycle (`helm install`, `helm upgrade`, `helm diff`).

### 5.3 Resource Requests & Limits
- Define CPU/memory requests (minimum guaranteed) and limits (maximum allowed).
- Proper sizing enables effective scheduling and cluster stability.
- Use `kubectl top` to validate sizing; adjust based on real usage.
- Understand QoS classes (Guaranteed, Burstable, BestEffort) and their scheduling implications.

### 5.4 Probes & Lifecycle Hooks
- Liveness, readiness, and startup probes control pod lifecycle and traffic routing.
- Lifecycle hooks (postStart, preStop) handle graceful shutdown.
- Simulate failures to observe Kubernetes restarting pods.

---

## 6. Storage

### 6.1 Volumes
- **emptyDir** – Ephemeral storage tied to Pod lifecycle.
- **hostPath** – Direct mount from node filesystem (not portable).
- **PersistentVolume** – Cluster-level storage resource.
- **CSI Drivers** – Modern interface for storage vendors; learn how to install one for your environment.

### 6.2 PersistentVolumeClaims (PVC)
- Pods request storage via PVCs; Kubernetes binds to a matching PV.
- Inspect bindings: `kubectl get pv,pvc`.
- Understand access modes (ReadWriteOnce/Many, ReadOnlyMany).

### 6.3 StorageClasses & Dynamic Provisioning
- StorageClasses define provisioners and parameters.
- With dynamic provisioning, PVCs automatically create PVs.
- Experiment with reclaim policies (Retain, Delete, Recycle).
- Use snapshots (if supported) for point-in-time copies.

### 6.4 Stateful Applications
- Use StatefulSets + PVC templates.
- Consider backup/restore strategies.
- Practice scaling StatefulSets and observe deterministic naming (`pod-0`, `pod-1`).
- Explore operators (e.g., for databases) that manage complex stateful workloads.

---

## 7. Scaling and Updates

### 7.1 Horizontal Pod Autoscaler (HPA)
- Scales Pods based on metrics (CPU, custom metrics).
- `kubectl autoscale deployment hello --min=2 --max=5 --cpu-percent=60`
- Use load generators (e.g., `hey`, `wrk`) to trigger scaling events.

### 7.2 Vertical Pod Autoscaler (VPA)
- Suggests or automatically adjusts resource requests/limits.
- Install in a sandbox cluster to observe recommendation updates.

### 7.3 Cluster Autoscaler
- Adds/removes nodes (in cloud environments).
- Understand integration with node groups or machine pools.
- Learn about pod disruption budgets (PDBs) to protect workloads during scale events.

### 7.4 Rolling Updates & Rollbacks
- `kubectl rollout status deployment/hello`
- `kubectl rollout history deployment/hello`
- `kubectl rollout undo deployment/hello`
- Experiment with `maxUnavailable` and `maxSurge` strategy fields.
- Simulate failures to practice rollback procedures.

---

## 8. Security Fundamentals

### 8.1 RBAC (Role-Based Access Control)
- **Role/ClusterRole** – Define permissions.
- **RoleBinding/ClusterRoleBinding** – Attach roles to users/groups/service accounts.
- Audit: list permissions with `kubectl auth can-i`.

### 8.2 Service Accounts
- Identity for Pods; used for API access.
- Associate workloads with least-privilege service accounts.
- Understand projected service account tokens and IAM integration (IRSA/Workload Identity).

### 8.3 Network Policies
- Define allowed ingress/egress traffic between Pods.
- Deploy a default deny policy, then open only needed ports.
- Use tools like `kubectl exec` with `curl` or `netcat` to test connectivity.

### 8.4 Pod Security
- Pod Security Standards (Baseline, Restricted) or Admission Controllers.
- Use securityContext: run as non-root, drop capabilities, read-only root filesystem.
- Evaluate PSP replacements (OPA Gatekeeper, Kyverno) for policy enforcement.

### 8.5 Secret Management
- Integrate with external secret stores (e.g., HashiCorp Vault, AWS Secrets Manager).
- Explore the CSI Secrets Store driver for syncing secrets as volumes.
- Rotate secrets regularly and monitor for drift.

### 8.6 Supply Chain Security
- Scan container images (Trivy, Grype) before deploying.
- Sign images/charts and verify with tools like Cosign.
- Understand admission control (e.g., enforcing signed images).

---

## 9. Observability & Operations

### 9.1 Logging
- Centralize logs with EFK (Elasticsearch/Fluentd/Kibana) or Loki/Grafana stacks.
- Use structured logging to enrich observability.

### 9.2 Metrics
- Deploy Prometheus + Grafana; learn about scraping, relabeling, and alerting rules.
- Observe pod/node metrics and create custom dashboards.

### 9.3 Tracing
- Integrate OpenTelemetry or Jaeger for distributed tracing.
- Instrument services to understand request paths across microservices.

### 9.4 Health & Readiness Checks
- Monitor component statuses (`kubectl get componentstatuses` or metrics).
- Implement synthetic tests that validate key application flows.

### 9.5 Backup & Disaster Recovery
- Backup `etcd` regularly for control plane recovery.
- Snapshot persistent volumes; test restore processes.
- Plan for multi-region failover if using managed Kubernetes.

---

## 10. Advanced Topics & Next Steps

### 10.1 GitOps
- Use Flux or Argo CD to manage desired state from Git.
- Model environments as branches or directories; automate reconciliation.

### 10.2 Operators & Custom Resources
- Extend Kubernetes with CustomResourceDefinitions (CRDs).
- Explore the Operator pattern for domain-specific automation.
- Write a basic controller (e.g., with Kubebuilder) to deepen understanding.

### 10.3 Multi-Cluster & Federation
- Manage multiple clusters with tools like Rancher or Cluster API.
- Understand service discovery, networking, and policy propagation across clusters.

### 10.4 Cost Optimization
- Right-size node pools, leverage spot/preemptible nodes where appropriate.
- Use resource quotas and limit ranges per namespace.
- Track spend using cloud cost tools or custom metrics.

### 10.5 Continued Learning
- Follow the [Kubernetes blog](https://kubernetes.io/blog/) and release notes every quarter.
- Participate in CNCF webinars or local meetups.
- Contribute by filing issues, submitting PRs, or improving documentation.

---

## 11. Practice Resources

- Work through the [hands-on demo](demo/README.md) included in this repository for curated exercises.
- Try Kubernetes the Hard Way by Kelsey Hightower to understand cluster bootstrapping.
- Explore managed services (GKE, EKS, AKS, DigitalOcean Kubernetes) for production-oriented scenarios.
- Leverage labs on Katacoda/KubeAcademy/Play with Kubernetes for disposable environments.
- Attempt CNCF certification paths (CKA, CKAD, CKS) to validate your skills.

Document your journey—capture commands, errors, and resolutions. Kubernetes mastery grows with repetition and real-world experiments.

---


