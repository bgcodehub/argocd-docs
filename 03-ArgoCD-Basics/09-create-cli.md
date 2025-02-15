# Creating an ArgoCD Application Using the CLI

## Overview

In this section, we will learn how to create an **ArgoCD application** using the **CLI**. Previously, we created and managed an application using the **UI**, but now we will explore how to achieve the same process through command-line commands.

This guide will cover:
- **Creating an ArgoCD application via the CLI**
- **Synchronizing the application state**
- **Verifying deployment status**
- **Updating and rolling back deployments**
- **Best practices for using the CLI**

Each step is documented in detail with explanations and corresponding commands.

---

## Step 1: Creating an ArgoCD Application Using the CLI

Instead of using the UI, we will use the `argocd app create` command. This command requires a repository URL, application name, destination namespace, and destination cluster.

### Steps:
1. **Ensure you are logged into ArgoCD:**
   ```bash
   argocd login <argocd-server-ip>
   ```
2. **Create a new ArgoCD application:**
   ```bash
   argocd app create solar-system-app-2 \
     --repo https://gitea.example.com/gitops-argocd.git \
     --path solar-system \
     --dest-server https://kubernetes.default.svc \
     --dest-namespace solar-system
   ```
3. **Verify the application is created:**
   ```bash
   argocd app list
   ```

📌 *Ensure the repository contains the necessary Kubernetes manifests before creating the application.*

---

## Step 2: Synchronizing the Application

By default, new applications in ArgoCD are not automatically synced. We must manually trigger synchronization.

### Steps:
1. **Check the synchronization status:**
   ```bash
   argocd app get solar-system-app-2
   ```
2. **If the application is out of sync, synchronize it manually:**
   ```bash
   argocd app sync solar-system-app-2
   ```
3. **Monitor synchronization status:**
   ```bash
   argocd app list
   ```

📌 *Synchronization ensures that the live state matches the desired state defined in the Git repository.*

---

## Step 3: Verifying Deployment Status

Once the application is synced, verify that the resources are deployed and running.

### Steps:
1. **Check Kubernetes namespace for deployed resources:**
   ```bash
   kubectl get all -n solar-system
   ```
2. **Retrieve application details using ArgoCD CLI:**
   ```bash
   argocd app get solar-system-app-2
   ```
3. **If any resources are missing, troubleshoot using logs:**
   ```bash
   kubectl logs -l app=solar-system -n solar-system
   ```

📌 *Pods, services, and deployments should be created successfully in the target namespace.*

---

## Step 4: Updating an Application and Synchronizing Changes

Updating an application involves modifying its Kubernetes manifests in the Git repository and syncing the changes.

### Steps:
1. **Modify the application YAML file in the repository.**
   Example: Updating the image version in `deployment.yaml`
   ```yaml
   spec:
     containers:
       - name: solar-system
         image: myrepo/solar-system:v6
   ```
2. **Commit and push the changes to Git:**
   ```bash
   git commit -am "Updated image to v6"
   git push origin main
   ```
3. **Synchronize the application in ArgoCD:**
   ```bash
   argocd app sync solar-system-app-2
   ```
4. **Verify the new deployment:**
   ```bash
   kubectl get pods -n solar-system
   ```

📌 *Ensure the application state updates correctly after synchronization.*

---

## Step 5: Rolling Back to a Previous Version

If a deployment causes issues, we can roll back to a previous version using ArgoCD’s rollback feature.

### Steps:
1. **Check deployment history:**
   ```bash
   argocd app history solar-system-app-2
   ```
2. **Rollback to a previous known good version:**
   ```bash
   argocd app rollback solar-system-app-2 <revision-id>
   ```
3. **Verify rollback success:**
   ```bash
   argocd app get solar-system-app-2
   ```

📌 *Rollbacks use previous Git commit versions stored in ArgoCD.*

---

## Step 6: Deleting an Application

If you no longer need an application, remove it and its associated resources.

### Steps:
1. **Delete the application from ArgoCD:**
   ```bash
   argocd app delete solar-system-app-2
   ```
2. **Ensure all resources are removed from Kubernetes:**
   ```bash
   kubectl get all -n solar-system
   ```
3. **Manually delete the namespace if necessary:**
   ```bash
   kubectl delete namespace solar-system
   ```

📌 *Deleting an application removes all associated Kubernetes resources, but namespaces may need manual deletion.*

---

## Conclusion

| Feature | Status |
|---------|--------|
| Creating an ArgoCD application via CLI | ✅ |
| Synchronizing application state | ✅ |
| Verifying deployment status | ✅ |
| Updating and rolling back applications | ✅ |
| Deleting applications and cleaning up | ✅ |

