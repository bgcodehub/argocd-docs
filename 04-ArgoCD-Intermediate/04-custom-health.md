# Creating a Custom Health Check for ConfigMaps in ArgoCD

## Overview

In this section, we will walk through the process of creating a **custom health check** for **ConfigMaps** in ArgoCD. ArgoCD provides robust health assessment features, allowing users to define custom health checks for Kubernetes resources beyond the default status checks. This is particularly useful when dealing with **ConfigMaps**, which do not have built-in health statuses.

By the end of this section, you will understand:
- **What a custom health check is** and why it is needed.
- **How to define a custom health check** in ArgoCD.
- **Using Lua scripting** to create a health check for ConfigMaps.
- **Applying the health check in a live ArgoCD environment.**

---

## 1. Why Do We Need a Custom Health Check for ConfigMaps?

ConfigMaps in Kubernetes are commonly used to store **configuration data** for applications. However, **they lack an inherent health status**, meaning ArgoCD does not automatically detect issues with their contents.

For example, suppose we have an application where different shapes (square, oval, circle, and rectangle) are assigned colors using a ConfigMap. If a **triangle** shape is also defined but given a white color (which makes it invisible), this could introduce a misconfiguration.

A custom health check would allow us to:
- Detect misconfigured ConfigMaps (such as incorrect color assignments).
- Degrade the health status of an application when a ConfigMap contains incorrect values.
- Provide **clear messages** in ArgoCD’s UI about why the health check has failed.

---

## 2. Defining a Custom Health Check in ArgoCD

ArgoCD supports defining **custom health checks** through its `resource.customizations.health` field. These custom checks can be written using **Lua scripting**.

### Key Concepts:
- **Health Status:** Determines whether a resource is `Healthy`, `Degraded`, or `Progressing`.
- **Lua Scripting:** Used to define the health logic within ArgoCD.
- **Custom Fields:** Custom checks can analyze specific ConfigMap values.

---

## 3. Implementing a Custom Health Check for ConfigMaps

### Step 1: Define the Custom Health Check

To create a **custom health check** for ConfigMaps, add the following entry in ArgoCD’s ConfigMap (`argocd-cm`). This tells ArgoCD how to evaluate the health of ConfigMaps:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  resource.customizations.health.ConfigMap: |
    local hs = {}
    hs.status = "Healthy"
    hs.message = "ConfigMap is valid"

    if obj.data ~= nil and obj.data["triangle"] == "white" then
      hs.status = "Degraded"
      hs.message = "Use a different color for the triangle"
    end
    return hs
```

### Explanation:
- **Checks if a `triangle` key exists** in the ConfigMap.
- **If the triangle color is `white`, the status is changed to `Degraded`**.
- **If no issues are found, the ConfigMap is marked as `Healthy`**.

### Step 2: Apply the Custom Health Check
After modifying the `argocd-cm` ConfigMap, apply the changes:
```bash
kubectl apply -f argocd-cm.yaml
kubectl rollout restart deployment argocd-repo-server -n argocd
```

This will ensure the new health check logic is applied.

---

## 4. Testing the Custom Health Check

### Deploying a ConfigMap
Create a sample ConfigMap with a **misconfigured value**:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-config
  namespace: default
data:
  triangle: "white"
```
Apply it:
```bash
kubectl apply -f example-config.yaml
```

### Checking the Status in ArgoCD
- **Go to the ArgoCD UI**.
- Locate the application using the ConfigMap.
- Observe the **Degraded** status with the message: `Use a different color for the triangle`.
- Modify the ConfigMap to use a valid color:
```yaml
triangle: "red"
```
- **Synchronize the application** in ArgoCD.
- Observe the **Healthy** status.

---

## 5. Advanced Customization

To further refine the health check, you can:
- Add additional **validation rules** (e.g., ensure numeric values are within a range).
- Apply different **conditions for other ConfigMap keys**.
- Extend Lua logic to check against **multiple ConfigMaps** at once.

Example:
```lua
if obj.data["background"] == obj.data["triangle"] then
  hs.status = "Degraded"
  hs.message = "Triangle color must not match the background"
end
```

---

## 6. Summary

| Feature | Description |
|---------|-------------|
| **Custom Health Checks** | Allows ArgoCD to assess ConfigMap validity. |
| **Lua Scripting** | Defines custom rules to evaluate ConfigMap contents. |
| **Health Status Updates** | Automatically updates ArgoCD UI based on ConfigMap values. |
| **Error Messages** | Displays meaningful messages when a misconfiguration is detected. |

By implementing **custom health checks** in ArgoCD, you can ensure better **configuration validation**, **error detection**, and **proactive issue resolution** in your Kubernetes environments.

---

## 7. Next Steps
- **Try modifying** the Lua script to include other misconfigurations.
- **Experiment** with different ConfigMap structures.
- **Use this approach** for other Kubernetes resources beyond ConfigMaps.

This method **enhances observability**, prevents **silent misconfigurations**, and **automates error detection** within your GitOps workflow.

