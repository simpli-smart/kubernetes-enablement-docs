# Cluster Tools

Simplismart provides pre-configured tools that can be installed on your clusters to enhance functionality.

---

## Overview

Cluster tools extend your cluster's capabilities with:

- **Monitoring**: Prometheus, Grafana for observability
- **Async Processing**: Redis, RabbitMQ for async inference pipelines
- **Training**: Tools for model training workloads

---

## Installing Tools

### During Cluster Import

Specify tools when importing a cluster:

```json
{
  "cluster_tools": {
    "TRAINING": ["uuid-of-training-tool"],
    "ASYNC TOOLS": ["uuid-of-redis", "uuid-of-rabbitmq"],
    "MONITORING": ["uuid-of-prometheus-stack"]
  }
}
```

### After Cluster Import

Currently, you can only add tools to a cluster during import. Contact support team to add tools to an existing cluster.

---

## Available Tool Categories

### Monitoring Stack

| Tool | Description |
|------|-------------|
| **Prometheus** | Metrics collection and storage |
| **Grafana** | Metrics visualization and dashboards |
| **DCGM Exporter** | NVIDIA GPU metrics |
| **Kubernetes Event Exporter** | Kubernetes event monitoring |
| **Mimir** | Metrics storage and querying |
| **Loki** | Logging storage and querying |
| **Promtail** | Logging collection |
| **SS Agent** | Simplismart agent for proactive health monitoring and reconciliation |

**Required for**: Full observability, autoscaling based on custom metrics

### Async Tools

| Tool | Description |
|------|-------------|
| **Redis** | In-memory data store for caching and queues |
| **RabbitMQ** | Message broker for async processing |

**Required for**: Async deployment type (`async_deployment_type: "ASYNC"`)

### Training Tools

| Tool | Description |
|------|-------------|
| **Ray Job Manager** | ML workflow orchestration |
| **Kuberay Operator** | Distributed training jobs |
| **Kuberay API Server** | Ray job management API |
| **Training Models Prepuller** | Pre-pull training models |

**Required for**: Model training workloads

### Scaling

| Tool | Description |
|------|-------------|
| **Metrics Server** | Metrics server |
| **Cluster Autoscaler** | Cluster autoscaler for Node Scaling |
| **Prometheus Adapter** | Prometheus adapter for custom metrics |
| **KEDA** | Kubernetes Event-driven Autoscaler |

**Required for**: Autoscaling based on custom metrics, event-driven autoscaling

---

## Tool Configuration

### Monitoring Configuration

The monitoring stack is pre-configured with:

- Retention: 90 days default
- Scrape interval: 30 seconds
- GPU metrics enabled (DCGM)

### Redis Configuration

Default Redis configuration:

| Setting | Value |
|---------|-------|
| Memory limit | 2Gi |
| Persistence Size | 60Gi |
| HA Mode | Available |

### RabbitMQ Configuration

Default RabbitMQ configuration:

| Setting | Value |
|---------|-------|
| Memory limit | 2Gi |
| Persistence Size | 60Gi |
| Management UI | Enabled |
| HA Mode | Available |

---

## Monitoring with Installed Tools

### Accessing Grafana

Once the monitoring stack is installed:

1. Navigate to your cluster in Simplismart
2. Access the Grafana link via `grafana.<cluster-fqdn>`
3. Use provided credentials

### Available Dashboards in UI

| Dashboard | Metrics |
|-----------|---------|
| Cluster Overview | Node health, resource usage |
| Deployment Metrics | Pod status, request rates |
| GPU Metrics | Utilization, memory |
| Node Metrics | CPU, memory, disk, network |

### Custom Dashboards

Create custom dashboards using:

- Prometheus metrics
- DCGM GPU metrics
- Custom application metrics

---

## Related Documentation

- [Import Cluster](../clusters/import-cluster.md)
- [Monitoring](./monitoring.md)
- [BYOC Deployments](../deployments/byoc-deployments.md)
- [Autoscaling](./autoscaling.md)
