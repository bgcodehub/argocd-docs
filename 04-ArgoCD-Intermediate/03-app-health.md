# ArgoCD Application Health Checks

## Overview
ArgoCD continuously monitors the state of deployed Kubernetes resources and provides **health checks** to determine their current status. These health checks help ensure that applications are functioning as expected and allow for troubleshooting when issues arise. This guide will cover:

- The different health statuses in ArgoCD
- Built-in health checks for Kubernetes resources
- How to create custom health checks

By the end of this guide, you should have a solid understanding of how ArgoCD assesses application health and how to configure it effectively.

---

## Understanding ArgoCD Health Statuses

ArgoCD assigns different health statuses to Kubernetes resources based on their condition. Below are the key statuses:

| **Health Status**  | **Description**  |
|--------------------|----------------|
| **Healthy**       | All resources are functioning correctly and meeting desired conditions. |
| **Progressing**   | Resources are not yet fully operational but are expected to become healthy. |
| **Degraded**      | A resource has failed or cannot reach a healthy state in time. |
| **Missing**       | The resource is not found in the cluster. |
| **Suspended**     | The resource is paused or inactive. |
| **Unknown**       | ArgoCD cannot determine the health status due to missing information. |

📌 *Health statuses are essential for determining if an application is running as expected. Issues like failing pods, misconfigured deployments, or missing services can trigger degraded or missing statuses.*

---

## Built-in Health Checks in ArgoCD

ArgoCD provides **default health checks** for various Kubernetes resources. These checks validate key attributes of resources and provide appropriate status updates. 

### Common Built-in Health Checks

| **Resource Type** | **Check Performed** |
|-------------------|-------------------|
| **Services (Load Balancer)** | Verifies that the load balancer has a valid hostname or IP. |
| **Ingress** | Checks that at least one valid hostname or IP exists. |
| **PersistentVolumeClaims (PVCs)** | Ensures that the PVC is correctly bound to a volume. |
| **Deployments, ReplicaSets, StatefulSets, DaemonSets** | Confirms that the observed generation and the desired generation match and that the correct number of replicas exist. |
| **ConfigMaps** | Ensures that the ConfigMap exists but does not check for conditions inside it. |

📌 *If a resource fails any of these checks, ArgoCD will report a degraded or missing status.*

---

## Creating Custom Health Checks

While ArgoCD provides built-in health checks, some Kubernetes resources require **custom health checks** to fit specific use cases. ArgoCD supports writing **custom health checks in Lua**, a lightweight scripting language.

### Example: Custom Health Check for a ConfigMap
Let’s assume we deploy a front-end application where the **background color** is defined in a **ConfigMap**. If a shape (e.g., a triangle) is set to white (matching the background), it will not be visible to users. We can create a custom health check to prevent this issue.

### Steps:
1. Define a health check in the **ArgoCD ConfigMap**.
2. Write a Lua script to check the condition.
3. If the issue is detected, set the status to **Degraded** with a custom message.

#### Example Lua Script:
```lua
health_status = {}
if obj.data["triangle_color"] == "white" then
  health_status.status = "Degraded"
  health_status.message = "Triangle color is set to white, making it invisible."
else
  health_status.status = "Healthy"
end
return health_status
```

📌 *This script ensures that if the triangle is set to white, ArgoCD will mark the ConfigMap as **Degraded**, alerting administrators to the issue.*

### Custom Health Checks for Other Kubernetes Resources
In addition to ConfigMaps, custom health checks can be created for:
- **CronJobs**
- **RuntimeClasses**
- **Roles and RoleBindings**
- **Secrets**
- **Namespaces**
- **Pods**
- **PVCs**

Using custom health checks simplifies **troubleshooting** and ensures resources are functioning correctly.

---

## Benefits of Health Checks
- **Proactive Monitoring:** Detect issues before they impact production.
- **Automated Troubleshooting:** Helps pinpoint misconfigurations quickly.
- **Improved Reliability:** Ensures that applications are functioning as intended.

📌 *If a resource is consistently unhealthy, checking ArgoCD's health status can quickly identify the root cause instead of manually inspecting all resources.*

---

## Conclusion
ArgoCD provides built-in and customizable health checks to maintain the reliability of deployed applications. By leveraging both **default** and **custom** health checks, teams can automate issue detection, improve debugging efficiency, and ensure application stability.

| **Feature** | **Supported in ArgoCD** |
|------------|------------------------|
| Built-in health checks for common resources | ✅ |
| Custom health checks using Lua scripting | ✅ |
| Health monitoring for all Kubernetes resources | ✅ |
| Automated troubleshooting through degraded statuses | ✅ |

By implementing **health checks**, teams can gain **better visibility** into their Kubernetes applications, ensuring **high availability** and **proactive issue resolution**.