# UDS (Unified Defense Stack)

## Overview

| Component | Role |
|-----------|------|
| **Zarf** | A free, open-source CLI tool that packages applications and their dependencies (images, Helm charts, manifests) into a single compressed file that can be deployed into disconnected/air-gapped environments with zero internet access. Also includes a builtin registry, Git server, and K9s dashboard. |
| **UDS** | A DevSecOps platform built on top of Zarf that adds security baselines, compliance automation, and standardized deployment patterns for DoD environments. |
| **Pepr** | A Kubernetes controller framework that lets you write cluster automation and policy logic in TypeScript instead of complex YAML or shell scripts — used by UDS for enforcing security posture. |
| **Lula** | Validates that NIST 800-53 security controls are actually implemented and met in a running environment. |

## What UDS Core Includes
- **Istio** — service mesh for zero-trust networking between services
- **Pepr** — policy enforcement engine
- **Prometheus + Grafana** — monitoring stack
- **Loki** — log aggregation
- **Neuvector** — container security

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