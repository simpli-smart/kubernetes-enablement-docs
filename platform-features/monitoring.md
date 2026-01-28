# Monitoring & Observability

Simplismart provides comprehensive monitoring capabilities to track the health, performance, and resource utilization of your clusters and deployments.

---

## Overview

Monitoring in Simplismart operates at multiple levels:

| Level | Tool | Description |
|-------|------|-------------|
| **Platform UI** | In-app Metrics | Quick reference metrics directly in the Simplismart dashboard |
| **Deep Dive** | Grafana Dashboards | Full metrics visualization and exploration |
| **Programmatic** | Health API | Real-time health status via API |

---

## In-Platform Metrics

The Simplismart dashboard provides at-a-glance metrics for each cluster and deployment:

### Available Deployment Metrics

| Metric | Description |
|--------|-------------|
| **Pod Status** | Number of ready vs. not-ready pods |
| **Health Status** | Overall deployment health (Healthy, Degraded, etc.) |
| **Resource Usage** | CPU, memory, GPU utilization |
| **Request Metrics** | Request count, latency, error rates |
| **Logs** | Logs for the deployment |
| **Events** | Events for the deployment |

#### Accessing Deployment Metrics

1. Navigate to **Deployments** in the Simplismart dashboard
2. Select your deployment
3. View the **Metrics** tab for real-time monitoring data
4. View the **Logs** tab for deployment logs
5. View the **Events** tab for deployment events

### Available Cluster Metrics

| Metric | Description |
|--------|-------------|
| **Node Status** | Number of ready vs. not-ready nodes |
| **Resource Usage** | CPU, memory, GPU utilization |
| **Node Details** | Node type, node pool, node status |
| **Pod Details** | Pod name, pod status |

#### Accessing Cluster Metrics

1. Navigate to **Clusters** in the Simplismart dashboard
2. Select your cluster
3. View the **Metrics** tab for real-time monitoring data

---

## Grafana Dashboards

For detailed analysis and historical data, Simplismart integrates with Grafana.

### Accessing Grafana

Grafana dashboards are available for clusters with the monitoring stack enabled and is reachable at `https://grafana.<cluster-fqdn>/` and requires authentication. Contact Simplismart team for access credentials.

---

## Health API

Programmatically check deployment health using the Health API.

### Endpoint

```
GET /deployments/model-deployment/fetch-status/{deployment_id}/
```

### Response

```json
{
  "data": "Healthy",
  "messages": [
    {
      "message": "Ready to use, the model is running and available for inference.",
      "severity": "info"
    }
  ],
  "pods": {
    "ready": 3,
    "not_ready": 0
  }
}
```

See [Deployment Lifecycle](../deployments/lifecycle.md#get-deployment-health) for full API documentation.

---

## Health Status Reference

| Status | Description | Action |
|--------|-------------|--------|
| `Healthy` | All systems operational | None required |
| `Progressing` | Deployment is scaling or updating | Wait for completion |
| `Initializing` | Deployment is starting up | Wait for pods to become ready |
| `Degraded` | Some pods are unhealthy | Investigate pod logs |
| `Unhealthy` | Deployment is not functioning | Check configuration and logs |
| `SCALED_DOWN` | Deployment is paused | Start deployment if needed |
| `Unknown` | Status cannot be determined | Contact support |
| `Missing` | Health data unavailable | Check cluster connectivity |

---

## Setting Up Monitoring for Imported Clusters

### For BYOC Clusters

When importing a cluster, install the entire monitoring stack as cluster tooling. See [Importing a Cluster](../clusters/import-cluster.md) for more details.

```json
{
  "cluster_tools": {
    "MONITORING": ["prometheus-stack-uuid", "grafana-uuid"]
  }
}
```

See [Cluster Tools](./cluster-tools.md) for available monitoring tools.

---

## Related Documentation

- [Health Checks](./health-checks.md)
- [Autoscaling](./autoscaling.md)
- [Cluster Tools](./cluster-tools.md)
- [Deployment Lifecycle](../deployments/lifecycle.md)
