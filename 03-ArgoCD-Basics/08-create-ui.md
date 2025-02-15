# Creating and Managing ArgoCD Applications: A Comprehensive Guide

## Overview

This guide provides a **step-by-step** approach to creating and managing applications in **ArgoCD** using the **user interface**. It is designed for individuals who would like to follow along and understand the key concepts, including:

- **Creating an ArgoCD application** via the UI
- **Managing repositories and secrets**
- **Synchronizing application state**
- **Rolling back an application** in case of issues
- **Deleting an application** and understanding its impact

Each step includes detailed explanations, required commands, and corresponding images for reference.

---

## Step 1: Setting Up the Git Repository

Before creating an application, ensure that you have a **Git repository** containing the required Kubernetes manifests. In this guide, we use a **self-hosted Git service (Gitea)**, but you can use **GitHub, GitLab, BitBucket, or any other Git service**.

### Steps:

1. **Sign in to Gitea or your preferred Git service.**
2. **Ensure the repository exists.** In this case, we use a repository called `gitops-argocd`.
3. **Check that the repository contains the required manifests.**

📌 *Make sure your repository has YAML files for service definitions and deployments.*

---

## Step 2: Creating a New ArgoCD Application

ArgoCD applications define the **desired state** for Kubernetes resources, mapping the application source (Git repo) to a **target cluster and namespace**.

### Steps:

1. Navigate to the **ArgoCD UI**.
2. Click on `+ New App` to create a new application.
3. Enter the **Application Name** (e.g., `solar-system-app-1`).
4. Select the **ArgoCD Project** (default project if no custom projects exist).
5. Choose the **Sync Policy**:
   - **Manual:** Requires manual synchronization.
   - **Automatic:** ArgoCD automatically syncs changes.
6. **Select a Repository** (Use `gitops-argocd` or your chosen Git repository).
7. Choose the **Path** inside the repo where manifests are stored (e.g., `/solar-system`).
8. Define the **destination Kubernetes cluster** and **namespace** (e.g., `solar-system`).

📌 *Ensure that the **`destination namespace`** exists or enable **`Auto-Create Namespace`** to allow ArgoCD to create it automatically.*

---

## Step 3: Managing Secrets in ArgoCD

Secrets are used to securely store sensitive information such as **TLS certificates, repository credentials, and Kubernetes secrets**.

### Steps to Manage Secrets:

1. Navigate to the **ArgoCD namespace** in Kubernetes.
2. Run the following command to list secrets:
   ```bash
   kubectl get secrets -n argocd
   ```
3. To inspect a specific secret (e.g., repo credentials):
   ```bash
   kubectl get secret <secret-name> -n argocd -o yaml
   ```
4. Decode Base64-encoded secrets:
   ```bash
   echo '<base64_string>' | base64 --decode
   ```

📌 *ArgoCD stores repository authentication details as secrets within Kubernetes. Ensure proper RBAC controls are in place.*

---

## Step 4: Synchronizing an ArgoCD Application

Synchronization ensures the **live state** of the application matches the **desired state** defined in Git.

### Steps:

1. In the ArgoCD UI, locate your application (`solar-system-app-1`).
2. Notice the **status:**
   - **Missing** – Resources are not yet deployed.
   - **Out of Sync** – Resources differ from the Git repository.
3. Click **Synchronize** to manually sync.
4. Select resources to sync and click **Synchronize**.

📌 *If resources fail to sync due to missing namespaces, enable **`Auto-Create Namespace`**.*

---

## Step 5: Rolling Back an Application

Rolling back is useful when an application is deployed with incorrect configurations.

### Steps:

1. Open the **History and Rollbacks** section in the ArgoCD UI.
2. Review the **list of previous deployments**.
3. Click on **Rollback** to revert to a stable configuration.
4. Confirm the rollback and verify the application’s status.

📌 *Rollback uses the previous Git commit version stored in ArgoCD.*

---

## Step 6: Deleting an ArgoCD Application

When an application is deleted, all **associated Kubernetes resources** (except the namespace) are also removed.

### Steps:

1. Navigate to the **ArgoCD UI**.
2. Locate the application (`solar-system-app-1`).
3. Click on `Delete`.
4. Confirm deletion.
5. Verify that resources are removed:
   ```bash
   kubectl get all -n solar-system
   ```
6. The namespace itself **is not deleted** unless manually removed:
   ```bash
   kubectl delete namespace solar-system
   ```

📌 *Deleting an application does not remove secrets or namespace unless explicitly deleted.*

---

## Conclusion

| Feature | Status |
|---------|--------|
| Setting up a Git repository | ✅ |
| Managing secrets for authentication | ✅ |
| Creating and synchronizing applications | ✅ |
| Rolling back changes if issues arise | ✅ |
| Deleting applications and cleaning up resources | ✅ |

