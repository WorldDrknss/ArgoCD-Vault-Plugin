---
# ArgoCD Application - Vault Plugin Sidecar

**Experimental, Use at your own risk**

This repository contains an ArgoCD application configuration for deploying the **ArgoCD Vault Plugin Sidecar**, which enables seamless integration of HashiCorp Vault with ArgoCD. This setup allows for secure management of secrets used in your Kubernetes applications.

## Prerequisites

- **ArgoCD** installed and configured on your Kubernetes cluster.
- Access to the Kubernetes environment where the Vault Plugin Sidecar will be deployed.
- Ensure that both Git and Helm repositories are accessible.

## Overview

The ArgoCD application is set up to deploy the following:

1. **ArgoCD Vault Plugin** from the GitHub repository (`https://github.com/WorldDrknss/argocd-vault-plugin.git`).

### Key Features:
- Automatically creates the namespace `argocd` if it doesn't already exist.
- Deploys the necessary configuration files for the Vault Plugin integration.

## Application Configuration

The ArgoCD application configuration (`vault-plugin-sidecar-argocd.yaml`) is as follows:

```bash
# enable key-value engine
vault secrets enable kv-v2
# add the password to path kv-v2/argocd
vault kv put kv-v2/argocd password="argocd"
# add a policy to read the previously created secret
vault policy write argocd - <<EOF
path "kv-v2/data/argocd" {
  capabilities = ["read"]
}
EOF
```

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: argocd-vault-plugin-sidecar
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://github.com/WorldDrknss/argocd-vault-plugin.git
    targetRevision: main
    path: "."
    directory:
      exclude: '{values.yaml,LISCENSE,README.md,manifests/*,merge/*}'  # Exclude unnecessary files
    kustomize:
      resources:
        - manifests/cmp-plugin.yaml
        - manifests/role.yaml
        - manifests/role-binding.yaml
      patchesStrategicMerge:
        - merge/argocd-repo-server.yaml
  destination:
    namespace: argocd
    server: https://kubernetes.default.svc
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
