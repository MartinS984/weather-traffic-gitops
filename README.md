# Weather & Traffic IoT GitOps Infrastructure

This repository contains the declarative GitOps configuration for a local Kubernetes infrastructure designed to ingest, stream, and monitor IoT weather and traffic telemetry.

## 🏗️ Architecture Stack
* **Platform:** Local Kubernetes via Minikube (WSL2/Ubuntu)
* **GitOps Engine:** ArgoCD (App of Apps pattern)
* **Data Backbone:** Strimzi Kafka Operator (KRaft mode)
* **Observability:** Kube-Prometheus-Stack & Loki (Grafana dashboards)

## 🚀 Deployment Phases Completed

### Phase 1: The GitOps Control Plane
* Deployed **ArgoCD** to manage cluster state declaratively from this repository.
* Implemented the **App of Apps** pattern via `minikube-root-infrastructure` to cascade updates to all child applications automatically.

### Phase 2: Observability Stack
* Deployed the **Prometheus & Grafana** stack to the `monitoring` namespace.
* Configured automated dashboards and data sources for cluster metrics and log aggregation.

### Phase 3: The Data Backbone (Kafka)
* Deployed the **Strimzi Kafka Operator** (v0.50.1).
* Provisioned a single-node, ZooKeeper-less **KRaft Kafka Broker** (v4.1.0) via `KafkaNodePool` resources.
* Provisioned the `iot-weather-data` Kafka topic (3 partitions, replication factor 1).

---

## 📖 SRE Runbook: Local Operations

Due to the nature of local Minikube deployments, aggressive host sleep states or Docker restarts can sever network tunnels or stall initializing pods. Use these commands to restore the control center.

### 1. Cluster Cold Boot
```bash
# Start the engine
minikube start
2. Restoring UI Tunnels (Port-Forwards)
Run these in separate background terminal tabs:

Bash
# ArgoCD UI (https://localhost:8080)
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Grafana UI (http://localhost:3000)
kubectl port-forward svc/prometheus-grafana -n monitoring 3000:80
3. Retrieving Credentials
ArgoCD Admin Password:

Bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode
Grafana Credentials: * Username: admin

Password: sre-admin-password

4. Clearing Stalled Pods (Kickstart)
If a pod is stuck in CrashLoopBackOff after a hard Minikube reset, delete the pod to let the ReplicaSet spawn a fresh one with clean networking.

Bash
# Kickstart Grafana
kubectl delete pod -l app.kubernetes.io/name=grafana -n monitoring

# Kickstart ArgoCD Repo Server (Fixes Sync Errors)
kubectl delete pods -l app.kubernetes.io/name=argocd-repo-server -n argocd