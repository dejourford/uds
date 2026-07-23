# UDS (Unified Defense Stack)

## Overview

| Component | Role |
|-----------|------|
| **Zarf** | A free, open-source CLI tool that packages applications and their dependencies (images, Helm charts, manifests) into a single compressed file that can be deployed into disconnected/air-gapped environments with zero internet access. Also includes a builtin registry, Git server, and K9s dashboard. |
| **UDS** | A DevSecOps platform built on top of Zarf that adds security baselines, compliance automation, and standardized deployment patterns for DoD environments. |
| **Pepr** | A Kubernetes controller framework that lets you write cluster automation and policy logic in TypeScript instead of complex YAML or shell scripts — used by UDS for enforcing security posture. |
| **Lula** | Validates that NIST 800-53 security controls are actually implemented and met in a running environment. |

## What UDS Core Slim Includes

| Namespace | Component | Role |
|-----------|-----------|------|
| `istio-system` | Istio (CNI, istiod, ztunnel) | Zero-trust service mesh — controls all traffic between services |
| `pepr-system` | Pepr controller + watcher | Policy enforcement — intercepts and validates cluster operations in real time |
| `keycloak` | Keycloak + waypoint | Identity provider — SSO and authentication for all UDS services |
| `authservice` | authservice | Handles authentication for Istio-protected services |
| `zarf` | Registry + agent-hook | Builtin OCI registry and mutating webhook for air-gapped image management |
| `istio-admin-gateway` | Admin ingress gateway | Ingress for admin/platform traffic |
| `istio-tenant-gateway` | Tenant ingress gateway | Ingress for application/tenant traffic |
| `uds-dev-stack` | MinIO | Object storage for the dev stack |
| `kube-system` | MetalLB, CoreDNS, nginx | Load balancing, DNS, and base cluster networking |

> **Note:** The full `k3d-core-demo` bundle additionally includes Prometheus, Grafana, Loki, and Neuvector but requires 16GB+ RAM. The slim bundle above runs on 8GB.

## Key Concepts
- **UDS Package** — a Zarf package + security declarations (network policies, SSO config)
- **UDS Bundle** — a collection of UDS packages deployed together
- **Zarf tarball** — self-contained bundle that can be physically transferred into an air-gapped network

---

## Setup

### Prerequisites
1. Install [Docker Desktop](https://www.docker.com/products/docker-desktop/)
2. Install [Homebrew](https://brew.sh/)

### Install k3d
```bash
brew install k3d
```

### Verify Prerequisites
```bash
docker info
k3d version
```

### Install the UDS CLI
```bash
brew install uds
```

### Verify UDS Installation
```bash
uds version
```

---

## Local Development

### Create a Local Cluster
```bash
k3d cluster delete uds-dev 2>/dev/null || true
k3d cluster create uds-dev --k3s-arg "--disable=traefik@server:*"
kubectl config use-context k3d-uds-dev
kubectl config current-context
```

### Verify the Cluster is Running
```bash
kubectl get nodes
```

### Deploy the UDS Core Slim Bundle
```bash
uds deploy k3d-core-slim-dev:latest
```

Confirm `y` when prompted. This deploys a lightweight UDS Core stack (Istio, Pepr) optimized for local development on machines with 8GB RAM. The full `k3d-core-demo` bundle requires 16GB+.

Takes 5-10 minutes.

### Watch the Rollout
```bash
uds zarf tools monitor
```

This opens K9s — a terminal UI for watching pods come up across all namespaces.

### Confirm UDS Core Slim Bundle is healthy
```
uds zarf tools kubectl get pods -A --no-headers | grep -Ev '(Running|Completed)'
```

No output means all pods are healthy.

## Adding a package

### Create a working directory
```
mkdir podinfo-package && cd podinfo-package
```

### Create UDS Package CR
```
apiVersion: uds.dev/v1alpha1
kind: Package
metadata:
  name: podinfo
  namespace: podinfo
spec:
  network:
    expose:
      - service: podinfo
        selector:
          app.kubernetes.io/name: podinfo
        gateway: tenant
        host: podinfo
        port: 9898
  sso:
    - name: Podinfo SSO
      clientId: uds-core-podinfo
      redirectUris:
        - "https://podinfo.uds.dev/login"
      enableAuthserviceSelector:
        app.kubernetes.io/name: podinfo
      groups:
        anyOf:
          - "/UDS Core/Admin"
  monitor:
    - selector:
        app.kubernetes.io/name: podinfo
      targetPort: 9898
      portName: http
      description: "podinfo metrics"
      kind: PodMonitor
```

### Create Zarf.yaml
```
kind: ZarfPackageConfig
metadata:
  name: podinfo
  version: 0.0.1

components:
  - name: podinfo
    required: true
    charts:
      - name: podinfo
        version: 6.10.1
        namespace: podinfo
        url: https://github.com/stefanprodan/podinfo.git
        gitPath: charts/podinfo
    manifests:
      - name: podinfo-uds-config
        namespace: podinfo
        files:
          - podinfo-package.yaml
    images:
      - ghcr.io/stefanprodan/podinfo:6.10.1
```

### Build and deploy the package
```
uds zarf package create --confirm
uds zarf package deploy zarf-package-podinfo-*.tar.zst --confirm
```

### Verify
```
uds zarf tools kubectl get package -n podinfo
```

The expected output is a pod named "podinfo" with the "Ready" status.

## Access the app
Navigate to 'https://podinfo.uds.dev' and you'll be redirected to keycloak. Only memners of /UDS Core/Admin can login.

### Create a test user by setting up a tasks.yaml file that imports a helper from uds-common:
```
includes:
  - common-setup: https://raw.githubusercontent.com/defenseunicorns/uds-common/main/tasks/setup.yaml
```

### Login to the app using the development creds
```
username: doug
password: unicorn123!@#UN
```
