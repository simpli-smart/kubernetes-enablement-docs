# Private Deployments API

Deploy containers and models on Simplismart's managed cloud infrastructure.

## Overview

Private Deployments run on Simplismart's managed cloud, where:

- **Automatic resource scheduling**: The platform selects optimal clusters and node groups
- **GPU optimization**: Intelligent scheduling based on GPU availability
- **No infrastructure management**: Focus on your models, not Kubernetes

This is the fastest way to get started with Simplismart.

---

## Create a Private Deployment

Deploy a container to Simplismart's managed cloud.

**Endpoint:** `POST /deployments/private/deploy-model/`

**Base URL:** `https://api.app.simplismart.ai`

**Authentication:** Bearer token
| Field | Type | Description |
|-------|------|-------------|
| `Authorization` | string | Bearer token |

**Example:**
```bash
curl -X POST "https://api.app.simplismart.ai/deployments/private/deploy-model/" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{ ... }'
```

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `model_repo` | UUID | Container UUID |
| `name` | string | Deployment name |
| `gpu_id` | string | GPU type (e.g., `nvidia-h100`, `nvidia-l4`, `cpu`) |
| `min_pod_replicas` | integer | Minimum replicas (≥ 1) |
| `max_pod_replicas` | integer | Maximum replicas (≥ 1) |
| `autoscale_config` | object | Autoscaling configuration |

### Optional Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `deployment_tag` | string | `production` | Environment tag (`testing`, `staging`, `production`) |
| `fast_scaleup` | boolean | `false` | Enable aggressive scale-up (requires org feature flag) |
| `persistent_volume_claims` | array | — | PVC configuration (only for custom models and imported containers) |
| `env_variables` | object | — | Environment variables (only for imported containers) |
| `deployment_custom_configuration` | object | — | Command/runtime overrides (only for imported containers) |
| `healthcheck` | object | Pod health check configuration (only for imported containers) |
| `ports` | object | Exposed service ports (only for imported containers) |
| `advanced_configuration` | object | Advanced configuration (only for async deployments) |

### Available GPUs

| GPU ID | Description |
|--------|-------------|
| `nvidia-h100` | NVIDIA H100 - Best for large LLMs |
| `nvidia-a100` | NVIDIA A100 - High-performance training/inference |
| `nvidia-l4` | NVIDIA L4 - Cost-effective inference |
| `nvidia-a10g` | NVIDIA A10G - Balanced performance |
| `nvidia-tesla-t4` | NVIDIA T4 - Entry-level GPU |
| `cpu` | CPU only (when supported by container) |

---

## Autoscaling Configuration

Autoscaling is **required** for all private deployments.

### Schema

```json
{
  "autoscale_config": {
    "targets": [
      {
        "metric": "cpu",
        "target": 80
      }
    ],
    "cooldown_period": 300
  }
}
```

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

### Validation Rules

- `autoscale_config` must be present and non-empty
- `targets` must be a non-empty array
- Each target must have:
  - `metric`: One of `cpu`, `gpu`, `ram`, `gram`, `latency`, `throughput`, `concurrency`, `queue_length`
  - `target`: An integer
  - `percentile`: Optional and is used for latency metrics (p50: 50, p75: 75, p90: 90, p95: 95)
  - `queue_length`: Only available for async deployments
  - `cooldown_period`: Only available for scale to zero deployments

### Invalid Examples

```json
// ❌ Missing targets
{ "autoscale_config": {} }

// ❌ Empty targets
{ "autoscale_config": { "targets": [] } }

// ❌ Invalid metric
{ "autoscale_config": { "targets": [{ "metric": "latency", "target": 200, "percentile": 95 }] } }

// ❌ Invalid percentile
{ "autoscale_config": { "targets": [{ "metric": "latency", "target": 200, "percentile": 100 }] } }
```

---

## Health Check Configuration

**Required for all Private Deployments of imported containers**

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
| `initialDelaySeconds` | integer | No | `30` | Delay before first probe |
| `periodSeconds` | integer | No | `10` | Interval between probes |
| `timeoutSeconds` | integer | No | `5` | Probe timeout |

### Best Practices

- Use a lightweight endpoint (`/health`, `/ready`)
- Ensure the endpoint does **not** require authentication
- Match `port` with your HTTP service port
- Increase `initialDelaySeconds` for slow-starting models

---

## Port Configuration

**Required for all Private Deployments of imported containers**

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

- `http` is **mandatory**
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
      "size": "50"
    }
  ]
}
```

### Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `mount_path` | string | Yes | Path to mount in container |
| `size` | string | Yes | Size in Gi |

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

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `consumer_count` | integer | Number of parallel consumers attached to a pod (only applicable for async deployments) |
| `termination_grace_period_seconds` | integer | Termination grace period in seconds |

---

## Example: Create Private Deployment

```bash
curl -X POST "https://api.app.simplismart.ai/deployments/private/deploy-model/" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "my-llm-deployment",
    "model_repo": "ecd1c408-8610-4a14-a48e-6212a5a29a51",
    "gpu_id": "nvidia-h100",
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
    "healthcheck": {
      "path": "/health",
      "port": 8200
    },
    "ports": {
      "http": {
        "port": 8200
      },
      "grpc": {
        "port": 9000
      }
    },
    "deployment_tag": "production",
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
  "model_repo": "ecd1c408-8610-4a14-a48e-6212a5a29a51",
  "name": "my-llm-deployment",
  "slug": "od06wglqtq",
  "org": "54360212-2263-44c6-afaf-1333544854f7",
  "deployment_tag": "production",
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
  "persistent_volume_claims": [],
  "env_variables": {},
  "deployment_custom_configuration": {},
  "ports": {
    "http": { "port": 8200 },
    "grpc": { "port": 9000 }
  },
  "healthcheck": {
    "path": "/health",
    "port": 8200,
    "initialDelaySeconds": 30,
    "periodSeconds": 10,
    "timeoutSeconds": 5
  },
  "fast_scaleup": false,
  "model_endpoint": "grpc.od06wglqtq.ss-in.s9t.link"
}
```

> **Note:** The `model_endpoint` is your deployment's inference endpoint, available after successful deployment.

---

## Error Responses

### Model Deleted (400 Bad Request)

```json
{
  "detail": "Model repo has been deleted and cannot be accessed."
}
```

### GPU Not Available (400 Bad Request)

```json
{
  "error": "Selected GPU not available at the moment"
}
```

### Duplicate Name (400 Bad Request)

```json
{
  "non_field_errors": ["Deployment with this name already exists for this organization."]
}
```

### GPU Mismatch (400 Bad Request)

```json
{
  "non_field_errors": ["GPU accelerator cannot be used when accelerator count is 0"]
}
```

### Feature Not Enabled (400 Bad Request)

```json
{
  "fast_scaleup": ["fast_scaleup is not enabled for your organization."]
}
```

### Wallet Blocked (402 Payment Required)

```json
{
  "error": "Wallet is blocked"
}
```

---

## Typical Workflow

1. **Create container** → `POST /models/client/model-repos/`
2. **Verify container status** → Ensure `status: SUCCESS`
3. **Deploy** → `POST /deployments/private/deploy-model/`
4. **Use endpoint** → Send inference requests to `model_endpoint`

---

## Related Documentation

- [BYOC Deployments](./byoc-deployments.md) - Deploy on your own clusters
- [Deployment Lifecycle](./lifecycle.md) - Health checks, scaling, start/stop
- [Imported Containers](../containers/imported-containers.md) - Create containers
- [Autoscaling](../platform-features/autoscaling.md) - Deep dive into autoscaling
