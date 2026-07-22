# UDS (Unified Defense Stack)

OVERVIEW:
Zarf - Packages the application (kubernetes)
UDS - Wraps the packaged application with security declarations, network policies, and compliance metadata
Pepr - Enforces the declared security posture in real time
Lula - Validates that NIST 800-53 controls are actually met

## Setup:

## Install
1. Install Docker Desktop
2. Install Homebrew
3. Install K3d
```
brew install k3d
```

## Verify
```
docker info
```
```
k3d version
```

## Install the UDS CLI
```
brew trust defenseunicorns/tap && brew install uds
```

## Verify Installation
```
uds version
```

## Create local k3d cluster and switch to it
```
k3d cluster create uds-dev
kubectl config use-context k3d-uds-dev
kubectl config current-context
```

## Verify it's running
```
kubectl get nodes
```

## Deploy the k3d-core-demo bundle and confirm 'y' when prompted
```
uds deploy k3d-core-demo:latest
```

## Watch the rollout with K9s
```
uds zarf tools monitor
```