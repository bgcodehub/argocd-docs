# Configuring a Git Webhook for ArgoCD Synchronization

## Overview
This guide walks through the process of configuring a **Git Webhook** to automate synchronization between **ArgoCD** and a Git repository. By setting up a webhook, changes pushed to the repository will automatically trigger synchronization with ArgoCD, ensuring that Kubernetes deployments remain up to date.

## Prerequisites
Before proceeding, ensure you have:
- An **ArgoCD** instance running in a Kubernetes cluster.
- A **Git repository** containing deployment manifests.
- Administrator access to the Git repository to configure webhooks.
- A running **Nginx application** for testing deployment synchronization.

---

## Step 1: Creating an ArgoCD Application for the Webhook
1. Navigate to **ArgoCD UI** and click **New App**.
2. Provide the following details:
   - **Application Name**: `git-webhook`
   - **Project**: Use `default` or create a new one.
   - **Repository URL**: Link to the **GitOps ArgoCD repository**.
   - **Path**: Directory containing the **Nginx application deployment manifest**.
   - **Destination Namespace**: `webhook`
3. Enable **Auto-Create Namespace** (if the namespace does not exist).
4. Click **Create**.
5. Synchronize the application manually to verify deployment.

📌 *The application should deploy successfully in the `webhook` namespace.*

---

## Step 2: Modifying the Deployment for Testing Synchronization
1. Open the **Nginx deployment manifest** in the Git repository.
2. Modify the `replicas` count (e.g., change from `1` to `2`).
3. Commit and push the changes.
4. Observe that **ArgoCD does not immediately reflect the changes**.

📌 *By default, ArgoCD polls the repository every **3 minutes**. We will optimize this using a webhook.*

---

## Step 3: Configuring a Webhook in Git
### **GitHub Users**:
1. Navigate to your **GitHub repository**.
2. Go to **Settings > Webhooks > Add webhook**.
3. Enter the **Payload URL**:
   ```
   http://<argocd-server-url>/api/webhook
   ```
4. Select **Content Type**: `application/json`.
5. Choose **Just the push event**.
6. Click **Add Webhook**.

### **GitLab/Bitbucket Users**:
- Follow similar steps, ensuring that the webhook URL points to your **ArgoCD API server**.

📌 *Ensure ArgoCD is configured to accept webhook events.*

---

## Step 4: Testing Webhook Synchronization
1. Modify the **Nginx deployment manifest** again (e.g., change replicas from `2` to `3`).
2. **Commit and push the change** to the repository.
3. The webhook should immediately send an event to ArgoCD.
4. In ArgoCD UI, check the **synchronization status**.
5. The application should automatically sync to match the new Git state.

📌 *If synchronization does not trigger, check webhook delivery logs in Git and ensure ArgoCD’s API endpoint is accessible.*

---

## Step 5: Handling Webhook Issues
If the webhook does not function properly, check for the following:
- **SSL Verification Issues**: If using self-signed certificates, GitHub may block the webhook. Use an **insecure** mode in ArgoCD or configure trusted certificates.
- **ArgoCD API Accessibility**: Ensure that ArgoCD’s API is reachable from GitHub/GitLab.
- **Misconfigured Webhook URL**: Double-check the webhook **Payload URL**.
- **ArgoCD ConfigMap Issues**:
   ```bash
   kubectl edit cm argocd-cm -n argocd
   ```
   Ensure `webhook.gitHub` is enabled.

---

## Conclusion
Using Git webhooks significantly enhances ArgoCD’s synchronization efficiency, reducing the delay introduced by periodic polling. By following this guide, users can ensure their deployments remain **continuously in sync** with Git repository changes.

| Feature | Status |
|---------|--------|
| Created ArgoCD application | ✅ |
| Configured webhook in Git | ✅ |
| Tested synchronization via webhook | ✅ |
| Troubleshooted common webhook issues | ✅ |

📌 *By implementing webhooks, deployments become more responsive, minimizing the time between Git changes and live updates in Kubernetes.*

