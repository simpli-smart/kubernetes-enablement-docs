# BYOC Deployments API

Deploy containers and models on your own Kubernetes clusters (Bring Your Own Compute).

## Overview

BYOC (Bring Your Own Compute) deployments allow you to run workloads on your created/imported Kubernetes clusters. Unlike Private Deployments, you have full control over:

- **Cluster selection**: Choose which cluster to deploy on
- **Node group targeting**: Specify which node groups should run your workload
- **Resource allocation**: Define CPU, memory, and GPU requirements

---

## Create a BYOC Deployment

Deploy a container or model to your BYOC cluster.

**Endpoint:** `POST /deployments/model-deployments/`

**Base URL:** `https://api.app.simplismart.ai`

**Authentication:** Bearer token
| Field | Type | Description |
|-------|------|-------------|
| `Authorization` | string | Bearer token |

**Example:**
```bash
curl -X POST "https://api.app.simplismart.ai/deployments/model-deployments/" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{ ... }'
```

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `model_repo` | UUID | Container UUID (must have `status: SUCCESS`) |
| `name` | string | Deployment name (3–30 chars, must start with letter, unique per org) |
| `cluster` | UUID | Target BYOC cluster UUID |
| `nodegroups` | array[UUID] | Node groups within the cluster |
| `cpu_request` | integer | vCPU requested per pod (must be > 0) |
| `cpu_limit` | integer | vCPU limit per pod (must be ≥ `cpu_request`) |
| `memory_request` | integer | Memory requested per pod in Gi (must be > 0) |
| `memory_limit` | integer | Memory limit per pod in Gi (must be ≥ `memory_request`) |
| `min_pod_replicas` | integer | Minimum pod replicas (must be ≥ 1) |
| `max_pod_replicas` | integer | Maximum pod replicas (must be ≥ `min_pod_replicas`) |
| `autoscale_config` | object | Autoscaling policy (required) |

### Optional Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `deployment_tag` | string | `production` | Environment tag (`testing`, `staging`, `production`) |
| `scale_to_zero_enabled` | boolean | `false` | Allow autoscaler to scale to zero |
| `persistent_volume_claims` | array | — | Attach persistent storage (only for imported containers and custom models) |
| `env_variables` | object | — | Environment variables (only for imported containers) |
| `deployment_custom_configuration` | object | — | Container entrypoint/command overrides (only for imported containers) |
| `ports` | object | — | **Required for Imported Containers** |
| `healthcheck` | object | — | **Required for Imported Containers** |
| `async_deployment_type` | string | `SYNC` | `ASYNC` or `SYNC` (`ASYNC` is only available for custom models)|
| `advanced_configuration` | object | — | Advanced configuration (only for async deployments) |

> **Note:** `ASYNC` deployment type requires Redis and RabbitMQ tooling to be installed on the cluster.

---

## Autoscaling Configuration

Autoscaling is mandatory for all deployments. Define scaling behavior using metric targets.

### Supported Metrics

| Metric | Description |
|--------|-------------|
| `cpu` | Average CPU utilization (%) |
| `gpu` | Average GPU utilization (%) |
| `ram` | Average memory utilization (%) |
| `gram` | Average GPU memory utilization (%) |
| `latency` | Latency quantiles (p50, p75, p90, p95) (ms) |
| `throughput` | Average throughput (RPS) |
| `concurrency` | Average concurrency (requests per pod) |
| `queue_length` | Average queue length (active messages per pod) for async deployments |

### Schema

```json
{
  "autoscale_config": {
    "targets": [
      {
        "metric": "cpu",
        "target": 80
      },
      {
        "metric": "gpu",
        "target": 75
      },
      {
        "metric": "latency",
        "target": 500,
        "percentile": 95
      }
    ],
    "cooldown_period": 300
  }
}
```

### Rules

- `targets` must be a non-empty array
- `metric` must be one of: `cpu`, `gpu`, `ram`, `gram`, `latency`, `throughput`, `concurrency`, `queue_length`
- `target` is typically an integer
- `percentile` is optional and is used for latency metrics (p50: 50, p75: 75, p90: 90, p95: 95)
- `queue_length` is only available for async deployments
- `cooldown_period` is the time in seconds to wait before scaling down, only applicable for scale to zero deployments
---

## Health Check Configuration

**Required for Imported Containers**

Health checks determine when your deployment is ready to receive traffic.

### Schema

```json
{
  "healthcheck": {
    "path": "/health",
    "port": 8200,
    "initialDelaySeconds": 30,
    "periodSeconds": 10,
    "timeoutSeconds": 5
  }
}
```

### Fields

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `path` | string | Yes | — | HTTP endpoint for health probing |
| `port` | integer | Yes | — | Container port to probe |
| `initialDelaySeconds` | integer | No | `30` | Delay before first probe (≥ 0) |
| `periodSeconds` | integer | No | `10` | Interval between probes (≥ 1) |
| `timeoutSeconds` | integer | No | `5` | Probe timeout (≥ 1) |

### Best Practices

- Use a lightweight endpoint (`/health`, `/ready`)
- Ensure the endpoint does not require authentication
- Match port with your exposed service ports
- Increase `initialDelaySeconds` for slow-starting models

---

## Port Configuration

**Required for Imported Containers**

Define which ports your deployment exposes.

### Supported Protocols

| Protocol | Required | Description |
|----------|----------|-------------|
| `http` | Yes | HTTP traffic port |
| `grpc` | No | gRPC traffic port |

### Schema

```json
{
  "ports": {
    "http": {
      "port": 8200
    },
    "grpc": {
      "port": 9000
    }
  }
}
```

### Rules

- `http` configuration is **mandatory**
- `grpc` is optional
- Only `http` and `grpc` protocols are supported

---

## Persistent Volume Claims

Attach persistent storage for stateful workloads.

### Schema

```json
{
  "persistent_volume_claims": [
    {
      "mount_path": "/data",
      "size": "50",
      "name": "pvc-data",
      "access_mode": "ReadWriteMany"
    }
  ]
}
```

### Fields

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `mount_path` | string | Yes | — | Path to mount in container |
| `size` | string | Yes | — | Size in Gi |
| `name` | string | No | `pvc-<mount_path>` | PVC name |
| `access_mode` | string | No | `ReadWriteMany` | Access mode |

---

## Imported Container Configuration

Override container runtime behavior for imported containers.

### Schema

```json
{
  "deployment_custom_configuration": {
    "entrypoint": ["python", "-m", "app"],
    "command": ["--port", "8200"],
    "workingDir": "/app"
  }
}
```

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `entrypoint` | array | Container entrypoint (non-empty array) |
| `command` | array | Command arguments (optional) |
| `workingDir` | string | Working directory |

---

## Advanced Configuration

Advanced configuration for runtime overrides.

### Schema

```json
{
  "advanced_configuration": {
    "runtime_overrides": {
      "consumer_count": 3,
      "termination_grace_period_seconds": 45
    }
  }
}
```

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `consumer_count` | integer | Number of parallel consumers attached to a pod (only applicable for async deployments) |
| `termination_grace_period_seconds` | integer | Termination grace period in seconds |

---

## Example: Create BYOC Deployment

```bash
curl -X POST "https://api.app.simplismart.ai/deployments/model-deployments/" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model_repo": "8b63bd7e-fd49-4906-a4f4-09539da8f597",
    "name": "my-imported-container-deployment",
    "cluster": "1e99a854-26de-4c19-b6b6-68c7e03c39c3",
    "nodegroups": ["a1b2c3d4-e5f6-7890-abcd-ef1234567890"],
    "cpu_request": 2,
    "cpu_limit": 4,
    "memory_request": 8,
    "memory_limit": 16,
    "min_pod_replicas": 1,
    "max_pod_replicas": 3,
    "autoscale_config": {
      "targets": [
        {
          "metric": "gpu",
          "target": 80
        }
      ]
    },
    "deployment_tag": "production",
    "scale_to_zero_enabled": false,
    "ports": {
      "http": {
        "port": 8200
      }
    },
    "healthcheck": {
      "path": "/health",
      "port": 8200
    },
    "advanced_configuration": {
    "runtime_overrides": {
        "consumer_count": 3,
        "termination_grace_period_seconds": 45
      }
    }
  }'
```

### Success Response

**Status:** `201 Created`

```json
{
  "uuid": "1e99a854-26de-4c19-b6b6-68c7e03c39c3",
  "name": "my-imported-container-deployment",
  "slug": "ufqeorbvf",
  "org": "54360212-2263-44c6-afaf-1333544854f7",
  "model_repo": "8b63bd7e-fd49-4906-a4f4-09539da8f597",
  "cluster": "1e99a854-26de-4c19-b6b6-68c7e03c39c3",
  "node_groups": ["a1b2c3d4-e5f6-7890-abcd-ef1234567890"],
  "min_pod_replicas": 1,
  "max_pod_replicas": 3,
  "autoscale_config": {
    "targets": [
      {"metric": "gpu", "target": 80}
    ]
  }
}
```

### Error Responses

#### Duplicate Name (400 Bad Request)

```json
{
  "non_field_errors": ["Deployment with this name already exists for this organization."]
}
```

#### Missing Node Groups (400 Bad Request)

```json
{
  "nodegroups": "List of node groups to be deployed on is expected"
}
```

#### Invalid Resource Requests (400 Bad Request)

```json
{
  "non_field_errors": ["CPU request and limit must be greater than 0"]
}
```

---

## Related Documentation

- [Private Deployments](./private-deployments.md) - Deploy on Simplismart's managed cloud
- [Deployment Lifecycle](./lifecycle.md) - Health checks, scaling, start/stop
- [Import a Cluster](../clusters/import-cluster.md) - Set up your BYOC cluster
- [Autoscaling](../platform-features/autoscaling.md) - Deep dive into autoscaling configuration
