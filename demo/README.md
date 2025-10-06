# Kubernetes Hands-On Demo

This demo gives you an end-to-end walkthrough for deploying and operating a simple microservice stack on a local Kubernetes cluster (Docker Desktop, minikube, or kind). It complements the top-level learning guide by providing concrete manifests and commands you can run immediately.

## Demo Goals

* Deploy an application stack composed of a web API, a backing Redis cache, and supporting jobs.
* Showcase core Kubernetes objects: Namespace, Deployment, Service, ConfigMap, Secret, HorizontalPodAutoscaler, Job, CronJob, and StatefulSet.
* Demonstrate configuration management, health probes, resource management, networking, and persistent storage.

## Prerequisites

* A working local Kubernetes cluster (`kubectl cluster-info` should succeed).
* `kubectl` v1.23+.
* (Optional) Metrics Server installed for the HorizontalPodAutoscaler. On Docker Desktop it is included by default; for minikube run `minikube addons enable metrics-server`.

## Directory Structure

```text
demo/
├── README.md                # This file
└── manifests/
    ├── 00-namespace.yaml
    ├── 01-configmap.yaml
    ├── 02-secret.yaml
    ├── 03-deployment.yaml
    ├── 04-service.yaml
    ├── 05-hpa.yaml
    ├── 06-job.yaml
    ├── 07-cronjob.yaml
    ├── 08-pvc.yaml
    ├── 09-statefulset.yaml
    └── kustomization.yaml
```

## Running the Demo

### 1. Inspect the Manifests

Each manifest is heavily commented. Read through them to understand how the objects map to the concepts in the learning guide.

### 2. Apply Everything with Kustomize

```bash
kubectl apply -k demo/manifests
```

This command creates a dedicated namespace (`kube-demo`) and deploys the whole stack.

### 3. Verify the Deployment

```bash
kubectl get all -n kube-demo
kubectl get pods -n kube-demo -w
```

Wait until all Pods transition to `Running` (Deployments) or `Completed` (Job/CronJob).

### 4. Access the Web API

Port-forward the web service to your machine:

```bash
kubectl port-forward svc/web-service -n kube-demo 8080:8080
```

Then curl the endpoint:

```bash
curl http://localhost:8080/
```

The response includes the message stored in the ConfigMap and the API key sourced from the Secret to illustrate configuration injection.

### 5. Observe Autoscaling

Generate load so the HorizontalPodAutoscaler scales the deployment:

```bash
kubectl run load-generator \
  --rm -i --tty \
  --image=busybox:1.36 \
  --restart=Never \
  --namespace kube-demo \
  -- /bin/sh -c "while true; do wget -q -O- http://web-service:8080; done"
```

In another terminal, watch the HPA react:

```bash
kubectl get hpa -n kube-demo -w
```

### 6. Inspect Jobs and StatefulSet

* View the one-off Job logs: `kubectl logs job/data-migration -n kube-demo`
* Confirm the CronJob scheduled run: `kubectl get cronjob -n kube-demo`
* Check Redis persistence: `kubectl exec statefulset/redis -n kube-demo -- redis-cli INFO persistence`

### 7. Clean Up

```bash
kubectl delete -k demo/manifests
```

## Troubleshooting Tips

* If Pods stay in `Pending`, run `kubectl describe pod <name> -n kube-demo` to check for resource or PVC issues.
* For PVC problems in Docker Desktop, ensure the built-in storage class is available via `kubectl get storageclass`.
* Metrics Server issues? Verify with `kubectl top pods -n kube-demo`. If it fails, install or enable Metrics Server first.

Enjoy experimenting! Try editing the manifests (e.g., change replica counts, tweak environment variables, add probes) and re-run `kubectl apply -k demo/manifests` to see how Kubernetes reconciles the new desired state.
