# ArgoCD Installation Options

## Overview
ArgoCD provides two primary installation options: **multi-tenant** and **core**. The choice between these options depends on the user's requirements for multi-tenancy, high availability, and the level of access needed for managing deployments.

---

## Core Installation
A **core installation** is best suited for users who do not require multi-tenancy. This installation is **simpler to set up**, has fewer components, and installs a **minimal non-HA version** of each component.

- Excludes the API server and user interface.
- Suitable for individual users or single-cluster environments.
- Provides a lightweight installation with basic functionality.

---

## Multi-Tenant Installation
The **multi-tenant installation** is the preferred method for organizations and development teams that require **segregated access** and **scalability**. This installation is managed separately by a **platform team** and kept up to date.

### **Multi-Tenant Installation Models**
There are two variations available:

1. **High-Availability (HA) Installation**
   - Recommended for **production environments**.
   - Includes multiple replicas of critical components for fault tolerance.
   - Ensures high availability and resilience.
   
2. **Non-High Availability (Non-HA) Installation**
   - Suitable for testing and proof-of-concept deployments.
   - Provides a basic installation without redundancy.
   - Not recommended for production use.

---

## Installation Manifests
ArgoCD offers two primary installation manifests:

### **1. `install.yaml` Manifest**
- Provides **standard installation** with **cluster-admin access** to ArgoCD.
- Used when deploying applications **within the same cluster** where ArgoCD is running.
- If cluster credentials are provided, this installation allows deployments to **remote clusters** as well.

### **2. `namespace-install.yaml` Manifest**
- Provides installation with **namespace-level access only**.
- Suitable for cases where ArgoCD should rely on provided cluster credentials instead of having full administrative access.
- Recommended for scenarios where **multi-tenancy** is required.

---

## High-Availability (HA) Installation
For production environments, it is recommended to use the **high-availability installation**. This approach:

- Uses the same `install.yaml` and `namespace-install.yaml` manifests.
- Includes **multiple replicas** for core ArgoCD components.
- Provides enhanced reliability and fault tolerance.

---

## Helm-Based Installation
ArgoCD can also be installed using **Helm**, which provides a **community-managed** chart. By default, this method installs the **non-HA version** of ArgoCD.

**Steps to Install ArgoCD using Helm:**
```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm install argocd argo/argo-cd -n argocd --create-namespace
```

---

## CLI Setup for ArgoCD
After installing ArgoCD, interaction with the API server requires the **ArgoCD CLI**.

### **Steps to Install ArgoCD CLI:**
```bash
# Download the latest ArgoCD CLI binary
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64

# Move the binary to a directory in your PATH
chmod +x argocd-linux-amd64
sudo mv argocd-linux-amd64 /usr/local/bin/argocd

# Verify installation
argocd version
```

---

## Conclusion
The choice of **ArgoCD installation type** depends on the level of access, availability, and scalability needed.
- **Core Installation:** Lightweight and simple.
- **Multi-Tenant Installation:** Preferred for organizations.
- **HA Installation:** Ensures redundancy and reliability.
- **Helm-Based Installation:** Provides an alternative approach for deploying ArgoCD.

