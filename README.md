# Simplismart Platform Documentation

**AI Infrastructure Platform for end-to-end MLOps management**

Simplismart provides a comprehensive platform for deploying, managing, and scaling machine learning models, on Kubernetes infrastructure. Whether you're using Simplismart's managed cloud or bringing your own compute, this documentation covers everything you need to get started.

---

## Table of Contents

- [Overview](#overview)
- [Key Concepts](#key-concepts)
- [Platform Features](#platform-features)
- [API Reference](#api-reference)
- [Roadmap](#roadmap)

---

## Overview

Simplismart enables platform engineers to:

- **Optimize Models**: Convert model weights into production-ready, optimized containers through our Model CI pipeline
- **Import Containers**: Bring your own application containers from private registries (Docker Hub, Depot)
- **Manage Clusters**: Create new clusters or import existing Kubernetes clusters (BYOC - Bring Your Own Compute)
- **Deploy at Scale**: Deploy models or containers with intelligent autoscaling and resource management
- **Monitor & Operate**: Full observability with Grafana dashboards, in-platform metrics and logs, and proactive health checks

### Deployment Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| **Private Deployment** | Deploy on Simplismart's managed cloud infrastructure | Quick start, no infrastructure management |
| **BYOC Deployment** | Deploy on your own Kubernetes clusters | Full control, compliance requirements, existing infrastructure |

---

## Key Concepts

### Clusters

A cluster represents your Kubernetes infrastructure. You can either:
- Let Simplismart manage infrastructure for you (Private Deployment)
- Bring your own compute (BYOC)
    - Import your existing Kubernetes clusters
    - Create new Kubernetes clusters via Simplismart Platform

**Related Documentation:**
- [Import Cluster](./clusters/import-cluster.md)
- [Manage Node Groups](./clusters/node-groups.md)

### Model Repositories

Model repositories in Simplismart can be:

1. **Optimized Models**: Models processed through our Model CI pipeline that converts weights into optimized inference containers
2. **Imported Containers**: Your own Docker images from private registries (Docker Hub, Depot)

**Related Documentation:**
- [Imported Containers](./containers/imported-containers.md)

### Deployments

A deployment represents a running instance of a container on your cluster. Deployments include:
- Resource allocation (CPU, memory, GPU, etc.)
- Autoscaling configuration
- Health checks and monitoring
- Service endpoints

**Related Documentation:**
- [BYOC Deployments](./deployments/byoc-deployments.md)
- [Private Deployments](./deployments/private-deployments.md)
- [Deployment Lifecycle](./deployments/lifecycle.md)

### Node Groups

Node groups define the compute resources within a cluster—machine types, GPU configurations, scaling limits, and capacity settings.

---

## Platform Features

### Monitoring & Observability

Simplismart provides comprehensive monitoring capabilities:

- **In-Platform Metrics**: Quick reference metrics and logs directly in the Simplismart UI
- **Grafana Dashboards**: Full metrics visualization for your clusters and deployments

📖 [Learn more about Monitoring](./platform-features/monitoring.md)

### Autoscaling

Intelligent autoscaling based on multiple metrics:

| Metric | Description |
|--------|-------------|
| `cpu` | Average CPU utilization across pods |
| `gpu` | Average GPU utilization |
| `ram` | Average memory utilization |
| `gram` | Average GPU memory utilization |
| `latency` | Average latency (ms) |
| `throughput` | Average throughput (RPS) |
| `concurrency` | Average concurrency (requests per pod) |
| `queue_length` | Average queue length (active messages per pod) for async deployments |

Configure scale-to-zero for cost optimization, or set minimum replicas for always-on availability.

📖 [Learn more about Autoscaling](./platform-features/autoscaling.md)

### Health Checks & Reconciliation

- **Proactive Health Checks**: Automatic health monitoring of all deployments
- **Status Reporting**: Clear, actionable health statuses (Healthy, Progressing, Degraded, etc.)

📖 [Learn more about Health Checks](./platform-features/health-checks.md)

### Continuous Deployment

ArgoCD-based continuous deployment workflows for your deployments:

- Declarative deployment configurations
- Automatic sync and drift detection
- Rollback capabilities

📖 [Learn more about Continuous Deployment](./platform-features/continuous-deployment.md)

### Cluster Tooling

Pre-configured tools for your clusters:

- **Monitoring Stack**: Prometheus, Grafana for observability
- **Async Tools**: Redis, RabbitMQ for async inference pipelines
- **Training Tools**: For model training workloads

📖 [Learn more about Cluster Tools](./platform-features/cluster-tools.md)

---

## API Reference

**Base URL:** `https://api.app.simplismart.ai`

**Authentication:** All API requests require a JWT token. See [Authentication](./getting-started/authentication.md) for details.

### Quick Links

| Resource | Operations | Documentation |
|----------|------------|---------------|
| **Clusters** | Import, Configure | [Clusters API](./clusters/import-cluster.md) |
| **Node Groups** | Add, Configure | [Node Groups API](./clusters/node-groups.md) |
| **Containers** | Create, List, Delete | [Containers API](./containers/imported-containers.md) |
| **Deployments (BYOC)** | Create, List, Delete, Start/Stop | [BYOC Deployments API](./deployments/byoc-deployments.md) |
| **Deployments (Private)** | Create, List, Delete | [Private Deployments API](./deployments/private-deployments.md) |
| **Lifecycle** | Health, Scaling, Start/Stop | [Lifecycle API](./deployments/lifecycle.md) |

---

## Roadmap

The following features are planned for future releases:

### Coming Soon

| Feature | Description |
|---------|-------------|
| **SDK Support** | Official Python SDK for easier integration |
| **Alert Management** | Configurable alerts for deployment health, resource usage, and anomalies |
| **Custom HELM Support** | Deploy custom HELM charts alongside your models |

---

## Getting Started

1. **[Authentication](./getting-started/authentication.md)** - Set up your JWT token
2. **[Import a Cluster](./clusters/import-cluster.md)** - Import your existing Kubernetes cluster
3. **[Create a Container](./containers/imported-containers.md)** - Import your container image
4. **[Deploy](./deployments/byoc-deployments.md)** - Launch your deployment
5. **[Monitor](./platform-features/monitoring.md)** - Track health and performance

---

## Support

For questions, issues, or feature requests:
- **Documentation**: [docs.simplismart.ai](https://docs.simplismart.ai)
- **Support**: Contact Simplismart team

---

*This documentation is intended for platform engineers integrating with the Simplismart Platform.*
