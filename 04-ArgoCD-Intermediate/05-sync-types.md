# ArgoCD Synchronization Strategies

## Overview

ArgoCD provides several **synchronization strategies** that allow users to control how applications are synchronized between a **Git repository** (the desired state) and a **Kubernetes cluster** (the actual state). These strategies determine how and when changes in the Git repository are applied to the cluster.

In this section, we will cover:
- The different **synchronization modes** in ArgoCD.
- How **auto-sync, self-healing, and pruning** impact application state.
- Real-world **examples and use cases** for each strategy.

---

## 1. Understanding ArgoCD Synchronization

ArgoCD synchronizes the desired state from **Git** to the target **Kubernetes cluster**. When a change is detected in the Git repository, ArgoCD can **apply the changes automatically** or **wait for manual intervention**.

### Key Concepts:
- **Auto-Sync**: ArgoCD applies updates automatically when changes are detected in the Git repository.
- **Manual Sync**: Users must manually trigger synchronization through the UI or CLI.
- **Pruning**: If files are removed from Git, ArgoCD will also remove the corresponding resources from the cluster.
- **Self-Healing**: If changes are made directly to the cluster (outside Git), ArgoCD restores them to match the repository.

---

## 2. Auto-Sync vs. Manual Sync

### Auto-Sync Mode:
- Ensures that any updates in the Git repository are **automatically applied** to the Kubernetes cluster.
- If new manifests are added (e.g., a new `service.yaml` file), ArgoCD **detects and applies** them without user intervention.
- Ideal for **continuous deployment (CD)** workflows.

#### Example:
Assume we have an ArgoCD application tracking a **Git repository** containing:
- `deployment.yaml`
- `service.yaml`
- `configmap.yaml`

If **auto-sync is enabled**, any changes to these files in Git **immediately update** the Kubernetes cluster.

### Manual Sync Mode:
- ArgoCD detects changes but **requires a user to manually sync** via the UI or CLI.
- Useful when deploying **mission-critical updates** that require manual review.
  
#### Example:
- A new version of an application is pushed to Git.
- ArgoCD detects the change but **waits** for the user to click "Sync" in the UI or run:
  ```bash
  argocd app sync my-application
  ```

---

## 3. Auto-Pruning

Auto-pruning is a feature that ensures **deleted files in Git** are also **removed from the cluster**.

### Behavior:
- If **auto-pruning is enabled**, when a user deletes a Kubernetes manifest from the Git repository, ArgoCD **removes** the corresponding resource from the cluster.
- If **disabled**, deleted files remain in the cluster unless manually removed.

#### Example:
1. A `configmap.yaml` file is deleted from Git.
2. If **auto-pruning is enabled**, ArgoCD removes the ConfigMap from the cluster.
3. If **auto-pruning is disabled**, the ConfigMap remains unless manually deleted.

Enable auto-pruning in the CLI:
```bash
argocd app set my-application --auto-prune
```

---

## 4. Self-Healing

Self-healing ensures that **direct changes** to the cluster are **reverted** to match the Git repository.

### Behavior:
- If a user manually modifies or deletes a resource in the cluster using `kubectl`, ArgoCD **restores it** to match Git.
- If self-healing is disabled, changes made directly to the cluster persist until the next manual sync.

#### Example:
1. A user **edits** a Kubernetes **Deployment** using:
   ```bash
   kubectl edit deployment my-app
   ```
2. If **self-healing is enabled**, ArgoCD detects the change and **reverts** it to match Git.
3. If **self-healing is disabled**, the manual change remains.

Enable self-healing in the CLI:
```bash
argocd app set my-application --self-heal
```

---

## 5. Combining Sync Strategies

ArgoCD allows combining **auto-sync, auto-pruning, and self-healing** for flexible deployments.

| **Strategy**      | **Effect** |
|------------------|-----------|
| Auto-Sync       | Automatically applies updates from Git. |
| Auto-Pruning    | Deletes resources from the cluster when removed from Git. |
| Self-Healing    | Restores cluster resources to match Git when manually changed. |

For a **fully automated GitOps workflow**, enable all three:
```bash
argocd app set my-application --auto-sync --auto-prune --self-heal
```

---

## Conclusion

Understanding ArgoCD’s **synchronization strategies** is crucial for managing Kubernetes applications effectively.
- **Auto-sync** ensures continuous deployment.
- **Auto-pruning** prevents resource drift by removing deleted files.
- **Self-healing** guarantees that direct changes to the cluster do not persist.

By selecting the appropriate strategy (or combining them), teams can optimize their **GitOps workflows** to fit their deployment needs.

