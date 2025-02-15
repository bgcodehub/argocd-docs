# Creating a Custom ArgoCD Project

## Overview
This guide walks through the process of creating a **custom ArgoCD project** using both the **ArgoCD UI and CLI**. The goal is to establish project-level restrictions, manage repository access, define deployment destinations, and set deny lists for resource constraints.

By the end of this guide, you will be able to:
- Create and configure a **custom ArgoCD project**.
- Restrict access to specific **Git repositories and clusters**.
- Define **deployment destinations** and namespaces.
- Apply **deny lists** to restrict resource creation.
- Deploy an application using this custom project.

---

## Step 1: Checking the Default ArgoCD Project
Before creating a custom project, check the **default ArgoCD project** and its permissions.

### Steps:
1. Open the ArgoCD UI.
2. Navigate to `Projects`.
3. Select the **default project** and review its configuration.
4. Note that the default project allows:
   - Deployments to **any cluster and namespace**.
   - Connection to **any Git repository**.
   - **No resource blacklisting.**

📌 *Since the default project has no restrictions, we will create a custom project with specific constraints.*

---

## Step 2: Creating a Custom Project in ArgoCD UI
### Steps:
1. Open the **ArgoCD UI** and navigate to `Projects`.
2. Click on `Create New Project`.
3. Provide a **Project Name** (e.g., `special-project`).
4. Under **Source Repositories**, allow access to a **specific repository only** (e.g., `pod-metadata` repository).
5. Under **Destination Clusters**, restrict deployments to **the current cluster only**.
6. Under **Namespace Restrictions**, allow deployment to a specific namespace or set `*` to allow all.
7. Under **Deny List**, restrict specific resources such as `ClusterRole` to prevent excessive permissions.
8. Click `Save`.

📌 *This project now restricts deployments to a single cluster, limits repository access, and enforces resource constraints.*

---

## Step 3: Managing Project Configuration via CLI
You can also create and configure projects using the **ArgoCD CLI**.

### Steps:
1. List existing projects:
   ```bash
   argocd proj list
   ```
2. Create a new project:
   ```bash
   argocd proj create special-project --description "Project with restricted access"
   ```
3. Restrict source repositories:
   ```bash
   argocd proj allow-source special-project https://github.com/example/pod-metadata.git
   ```
4. Restrict cluster access:
   ```bash
   argocd proj allow-cluster special-project https://kubernetes.default.svc
   ```
5. Set deny list for `ClusterRole` creation:
   ```bash
   argocd proj deny-resource special-project rbac.authorization.k8s.io ClusterRole
   ```
6. Verify the project configuration:
   ```bash
   argocd proj get special-project
   ```

📌 *This ensures that only the `pod-metadata` repository is accessible, and `ClusterRole` creation is blocked.*

---

## Step 4: Creating an Application Under the Custom Project
Now, let's create an application **using the restricted project**.

### Steps:
1. Navigate to `Applications` in the ArgoCD UI.
2. Click `+ New App`.
3. Enter the **Application Name** (e.g., `special-pod-app`).
4. Select **special-project** as the target project.
5. Use the **pod-metadata repository** and select the correct manifest path.
6. Choose **manual sync policy**.
7. Define the **destination cluster and namespace** (e.g., `special-pod` namespace).
8. Click `Create`.

📌 *If the repository is not allowed, ArgoCD will return an error indicating the source is not permitted.*

---

## Step 5: Syncing and Troubleshooting Deployment Issues
Once the application is created, synchronize it to deploy the resources.

### Steps:
1. Open the **Application Details** page.
2. Click `Sync` to start synchronization.
3. If synchronization fails due to `ClusterRole` restrictions, uncheck `ClusterRole` in the resource list and try again.
4. Confirm that the application deploys successfully.

📌 *Deny-listed resources will fail to deploy unless explicitly permitted.*

---

## Step 6: Validating Deployment
To confirm that the application is running:

### Steps:
1. Check deployment status in the ArgoCD UI.
2. Use the following command to verify Kubernetes resources:
   ```bash
   kubectl get all -n special-pod
   ```
3. Confirm the **service endpoint** by checking the NodePort:
   ```bash
   kubectl get svc -n special-pod
   ```
4. Access the application UI by visiting the NodePort in your browser.

📌 *The deployed application should now be accessible with the restrictions in place.*

---

## Conclusion
| Feature | Status |
|---------|--------|
| Creating a restricted ArgoCD project | ✅ |
| Defining source repository restrictions | ✅ |
| Limiting cluster and namespace access | ✅ |
| Denying specific Kubernetes resources | ✅ |
| Deploying an application under the custom project | ✅ |

