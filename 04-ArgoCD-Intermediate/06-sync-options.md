# ArgoCD Synchronization Strategies

## Overview

ArgoCD provides different **synchronization strategies** to control how applications are deployed and maintained within a **Kubernetes cluster**. Synchronization ensures that the **desired state** (defined in Git) and the **actual state** (running in Kubernetes) remain aligned.

In this section, we will explore:
- The different synchronization strategies in ArgoCD.
- How ArgoCD manages synchronization under different configurations.
- The impact of **automatic synchronization, self-healing, and auto-pruning**.
- Best practices for choosing the right synchronization strategy.

---

## 1. Understanding Synchronization in ArgoCD

When ArgoCD detects a difference between the **desired state** (Git) and the **actual state** (Kubernetes), it can:
- **Perform an automatic sync** to apply the changes immediately.
- **Wait for manual intervention** where a user must trigger synchronization.
- **Use self-healing mechanisms** to restore deleted resources.
- **Apply auto-pruning** to remove deleted resources from Kubernetes.

Each of these behaviors depends on the synchronization policies set within ArgoCD.

---

## 2. Manual vs. Automatic Synchronization

### **Manual Synchronization**
- In manual mode, ArgoCD detects changes but does **not** apply them automatically.
- Users must manually trigger synchronization via the **UI, CLI, or API**.
- This provides more control but requires **human intervention**.

**Example Scenario:**
- A user updates the **replica count** in `deployment.yaml`.
- ArgoCD marks the application as **out of sync**.
- The user must manually trigger a **sync operation** for the update to apply.

### **Automatic Synchronization**
- In this mode, ArgoCD applies updates **automatically** when changes are detected in Git.
- This eliminates manual intervention and ensures rapid **deployment of changes**.
- Useful for **continuous deployment pipelines**.

**Example Configuration:**
```yaml
syncPolicy:
  automated:
    prune: false
    selfHeal: false
```
- Here, **automatic sync is enabled**, but **auto-pruning** and **self-healing** are disabled.
- New changes will be deployed, but deleted resources in Git won’t be removed from Kubernetes.

---

## 3. Auto-Pruning: Removing Deleted Resources

### **What is Auto-Pruning?**
- When enabled, **auto-pruning** removes Kubernetes resources **that no longer exist in Git**.
- Ensures that any manually created resources **outside of Git** are deleted.
- Prevents **configuration drift** between Git and Kubernetes.

### **Example Scenario Without Auto-Pruning:**
- A user deletes `service.yaml` from Git.
- ArgoCD detects the deletion but **does not** remove the Service from Kubernetes.
- The Service continues running unless removed manually.

### **Example Scenario With Auto-Pruning Enabled:**
- A user deletes `service.yaml` from Git.
- ArgoCD automatically removes the Service from Kubernetes.

**Example Configuration:**
```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: false
```
- Auto-pruning is enabled (`prune: true`), so **resources deleted in Git are also deleted in Kubernetes**.

---

## 4. Self-Healing: Restoring Deleted Resources

### **What is Self-Healing?**
- Ensures that if a Kubernetes resource is **manually modified or deleted**, ArgoCD restores it.
- Prevents unauthorized changes outside of Git.
- Essential for maintaining **GitOps compliance**.

### **Example Scenario Without Self-Healing:**
- A user manually deletes a `ConfigMap` from Kubernetes.
- ArgoCD does **not** restore it because self-healing is disabled.

### **Example Scenario With Self-Healing Enabled:**
- A user manually deletes a `ConfigMap` from Kubernetes.
- ArgoCD detects the deletion and **automatically recreates it**.

**Example Configuration:**
```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```
- Here, **self-healing is enabled**, ensuring deleted or modified resources are restored.

---

## 5. Demonstrating Synchronization Policies

Let’s assume we have a **Git repository** with Kubernetes manifests, and an ArgoCD application is set up to synchronize these manifests.

### **Scenario 1: Manual Synchronization**
1. A user updates **replica count** in `deployment.yaml`.
2. ArgoCD marks the application as **out of sync**.
3. The user must manually trigger a sync using:
   ```bash
   argocd app sync my-app
   ```
4. The update is applied to the cluster.

### **Scenario 2: Automatic Synchronization Without Auto-Pruning**
1. A user updates `service.yaml` in Git.
2. ArgoCD automatically applies the update.
3. The user deletes `configmap.yaml` from Git, but the ConfigMap **remains in Kubernetes** because auto-pruning is disabled.

### **Scenario 3: Automatic Synchronization With Auto-Pruning**
1. A user deletes `configmap.yaml` from Git.
2. ArgoCD automatically **removes the ConfigMap from Kubernetes**.

### **Scenario 4: Self-Healing Enabled**
1. A user manually **deletes** a `Deployment` from Kubernetes using `kubectl delete deployment my-app`.
2. ArgoCD detects the deletion and **automatically recreates the Deployment**.

---

## 6. Choosing the Right Synchronization Strategy

| Synchronization Mode | Auto-Pruning | Self-Healing | Use Case |
|----------------------|-------------|-------------|----------|
| **Manual** | ❌ | ❌ | Full manual control over deployments. |
| **Automatic Sync** | ❌ | ❌ | CI/CD pipelines where human approval is needed for deletions. |
| **Automatic Sync + Auto-Pruning** | ✅ | ❌ | Strict GitOps workflows ensuring no unmanaged resources remain. |
| **Automatic Sync + Self-Healing** | ❌ | ✅ | Environments where unauthorized changes must be prevented. |
| **All Enabled** | ✅ | ✅ | Full GitOps compliance with automated deployments and enforcement. |

---

## Conclusion

- ArgoCD provides **flexible synchronization strategies** for different operational needs.
- **Manual synchronization** gives fine-grained control, while **automatic sync** ensures continuous deployments.
- **Auto-pruning** removes resources deleted from Git, and **self-healing** restores resources deleted from Kubernetes.
- Choosing the right strategy depends on your **GitOps compliance requirements and operational workflows**.

By carefully configuring **synchronization, auto-pruning, and self-healing**, teams can ensure that Kubernetes applications stay **reliable, consistent, and aligned with Git**.

