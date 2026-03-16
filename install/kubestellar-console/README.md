# Installing Microcks via KubeStellar Console

## Overview

[KubeStellar Console](https://console.kubestellar.io?utm_source=github&utm_medium=pr&utm_campaign=cncf_outreach&utm_term=microcks) is a multi-cluster Kubernetes dashboard with guided install missions for CNCF projects. The Microcks mission automates the standard Helm-based installation with pre-flight checks, step-by-step commands, validation queries, and rollback instructions.

This guide covers how to use the console's guided mission to install Microcks `1.13.2` on any Kubernetes cluster.

## Prerequisites

- Kubernetes cluster (v1.17+) with `kubectl` configured
- [Helm](https://helm.sh/docs/intro/install/) v3 installed
- Cluster-admin or namespace-admin permissions

## Option 1: Use the Guided Mission (Interactive)

The guided mission walks through each step interactively, querying your cluster to verify success at each stage.

### Install KubeStellar Console

```bash
curl -sSL https://raw.githubusercontent.com/kubestellar/console/main/start.sh | bash
```

This starts the console locally, connecting to your current kubeconfig context. No cloud account or OAuth required.

### Open the Microcks Mission

Navigate to the [Microcks install mission](https://console.kubestellar.io/missions/install-microcks?utm_source=github&utm_medium=pr&utm_campaign=cncf_outreach&utm_term=microcks) in the console. The mission executes these steps:

1. **Add the Microcks Helm repository**
   ```bash
   helm repo add microcks https://microcks.io/helm
   helm repo update
   ```

2. **Install Microcks via Helm**
   ```bash
   helm install microcks microcks/microcks \
     --namespace microcks \
     --create-namespace \
     --version 1.13.2
   ```

3. **Validate the installation**
   ```bash
   kubectl get pods --namespace microcks
   ```
   Expected: all pods in `Running` state (microcks, microcks-postman-runtime, microcks-mongodb, keycloak).

4. **Access the Microcks UI**
   ```bash
   kubectl port-forward svc/microcks --namespace microcks 8080:8080
   ```
   Open `http://localhost:8080` in your browser. Default credentials: `admin` / `microcks123`.

Each step includes validation — the console queries your cluster after each command to verify pods are running, services are reachable, and CRDs are registered. On failure, it reads pod logs and events to suggest fixes.

## Option 2: Use the Commands Directly

The mission works as read-only documentation too — no cluster connection required to browse the steps. You can copy-paste the commands above into any terminal.

## Uninstall

```bash
helm uninstall microcks --namespace microcks
kubectl delete namespace microcks
```

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| `microcks-mongodb` pod in `Pending` | No PVC provisioner | Install a CSI driver or use `--set mongodb.persistent=false` |
| `keycloak` pod in `CrashLoopBackOff` | Missing database connection | Check Keycloak logs: `kubectl logs -n microcks -l app=keycloak` |
| Port-forward fails with "connection refused" | Pod not ready | Wait for all pods: `kubectl wait --for=condition=ready pod -l app=microcks -n microcks --timeout=120s` |

## Mission Source

The mission definition is an open-source JSON file in the [console-kb](https://github.com/kubestellar/console-kb/blob/master/solutions/cncf-install/install-microcks.json?utm_source=github&utm_medium=pr&utm_campaign=cncf_outreach&utm_term=microcks) repository. PRs to improve the Microcks mission steps, add validation checks, or update versions are welcome.

## References

- [Microcks official documentation](https://microcks.io/documentation/)
- [Microcks Helm chart on ArtifactHub](https://artifacthub.io/packages/helm/microcks/microcks)
- [KubeStellar Console](https://console.kubestellar.io?utm_source=github&utm_medium=pr&utm_campaign=cncf_outreach&utm_term=microcks) — multi-cluster Kubernetes dashboard with 186 install missions for CNCF projects
