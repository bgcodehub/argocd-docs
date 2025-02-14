# Installing ArgoCD and ArgoCD CLI

## Overview
ArgoCD is a declarative, GitOps continuous delivery tool for Kubernetes. This guide will walk through installing ArgoCD and its CLI, configuring it, and logging in for the first time.

---

## Installing ArgoCD

### Step 1: Create the ArgoCD Namespace
Before installing ArgoCD, create a namespace for it within your Kubernetes cluster.

```bash
kubectl create namespace argocd
```

### Step 2: Apply the ArgoCD Manifest
Download and apply the ArgoCD installation manifest:

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

This command will deploy all necessary services, including the ArgoCD API server, Redis, Dex (authentication server), and the notification controller.

### Step 3: Check Deployment Status
Verify that all ArgoCD components are running:

```bash
kubectl get pods -n argocd
```

Once all pods are in `Running` state, ArgoCD is successfully installed.

---

## Exposing the ArgoCD Server
By default, the ArgoCD server is accessible via a `ClusterIP` service. To access it externally, change it to `NodePort`:

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
```

Retrieve the external IP and port:

```bash
kubectl get svc argocd-server -n argocd
```

Use this IP and port to access the ArgoCD UI in a browser.

---

## Logging into ArgoCD
### Step 1: Retrieve the Admin Password
ArgoCD generates an initial password stored in a Kubernetes secret. Fetch it with:

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
```

### Step 2: Log into the UI
- Open a browser and navigate to `https://<ARGOCD_SERVER_IP>:<PORT>`.
- Use the username `admin` and the retrieved password to log in.
- Change the password after logging in for security.

---

## Installing the ArgoCD CLI
To manage ArgoCD from the command line, install the CLI:

### Step 1: Download and Install the CLI
For Linux:

```bash
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x argocd-linux-amd64
sudo mv argocd-linux-amd64 /usr/local/bin/argocd
```

For macOS:

```bash
brew install argocd
```

### Step 2: Verify the Installation
```bash
argocd version
```

---

## Logging into ArgoCD via CLI
### Step 1: Log in Using the CLI
Use the CLI to log in to the ArgoCD server:

```bash
argocd login <ARGOCD_SERVER_IP> --username admin --password <PASSWORD>
```

### Step 2: List Available Applications
```bash
argocd app list
```

At this stage, no applications are deployed yet. The next steps involve configuring and deploying applications using ArgoCD.

---

## Conclusion
You have successfully installed ArgoCD and its CLI, exposed the service, and logged in. This setup will enable you to manage Kubernetes applications declaratively with GitOps. The next step is to deploy applications and configure Git repositories for continuous deployment.

