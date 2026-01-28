# Node Groups API

Register and manage node groups on your imported Kubernetes clusters.

## Overview

Node groups define the compute resources available for deployments within a cluster. Each node group specifies:

- Machine types and sizes
- GPU configurations
- Scaling limits (min/max nodes)
- Capacity type (on-demand vs. spot)

For imported clusters, create node groups in your cloud's console or using their CLI/Terraform modules.
Label the node groups with `simplismart.ai/node-pool-name:<node-pool-name>` before registering them with Simplismart.

---

## Register Node Groups to a Cluster

Register additional node groups to an existing imported cluster.

**Endpoint:** `POST /infrastructure/node-groups/{cluster_id}/import-cluster`

**Base URL:** `https://api.app.simplismart.ai`

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `cluster_id` | UUID | Your cluster's UUID |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `org` | UUID | Yes | Must match the cluster's organization |
| `nodegroup_labels` | array | Yes | List of node group definitions |

### Node Group Definition

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Unique name within the cluster |
| `is_auxilliary` | boolean | Yes | Whether this is an auxiliary node group |
| `node_config` | object | No | Detailed node configuration |

### Node Config Fields

| Field | Type | Description |
|-------|------|-------------|
| `machine_type` | string | Instance type (e.g., `c5.xlarge`, `n1-standard-8`) |
| `capacity_type` | string | `ON_DEMAND` or `SPOT` |
| `min_node_count` | integer | Minimum number of nodes |
| `max_node_count` | integer | Maximum number of nodes |
| `desired_count` | integer | Initial/desired node count |
| `accelerator_type` | string | GPU type (optional) |
| `accelerator_count` | integer | GPUs per node (optional) |
| `vcpu` | integer | vCPU count |
| `memory` | integer | Memory in GB |

### Example: Add a CPU Node Group

```bash
curl -X POST "https://api.app.simplismart.ai/infrastructure/node-groups/1e99a854-26de-4c19-b6b6-68c7e03c39c3/import-cluster" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "org": "54360212-2263-44c6-afaf-1333544854f7",
    "nodegroup_labels": [
      {
        "name": "cpu-pool",
        "is_auxilliary": false,
        "node_config": {
          "machine_type": "c5.xlarge",
          "capacity_type": "SPOT",
          "min_node_count": 1,
          "max_node_count": 5,
          "desired_count": 2
        }
      }
    ]
  }'
```

### Example: Add a GPU Node Group

```bash
curl -X POST "https://api.app.simplismart.ai/infrastructure/node-groups/1e99a854-26de-4c19-b6b6-68c7e03c39c3/import-cluster" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "org": "54360212-2263-44c6-afaf-1333544854f7",
    "nodegroup_labels": [
      {
        "name": "h100-pool",
        "is_auxilliary": false,
        "node_config": {
          "machine_type": "p4d.24xlarge",
          "capacity_type": "ON_DEMAND",
          "min_node_count": 0,
          "max_node_count": 4,
          "desired_count": 1,
          "accelerator_type": "nvidia-h100",
          "accelerator_count": 8
        }
      }
    ]
  }'
```

### Success Response

**Status:** `200 OK`

```json
{
  "message": "NodeGroup added successfully"
}
```

### Error Responses

#### Not a Kubernetes Cluster (400 Bad Request)

```json
{
  "error": "Cluster is not a kubernetes cluster"
}
```

**Solution:** This endpoint only works with imported Kubernetes clusters (BYOC).

#### Cluster Not Found (404 Not Found)

```json
{
  "error": "Cluster does not exist"
}
```

**Solution:** Verify the cluster UUID in the URL.

#### Duplicate Node Group (409 Conflict)

```json
{
  "error": "NodeGroup with name 'cpu-pool' already exists in this cluster."
}
```

**Solution:** Use a unique name for your node group.

#### Validation Failed (422 Unprocessable Entity)

```json
{
  "nodegroup_labels": ["<validation errors>"]
}
```

**Solution:** Check your node group configuration matches the expected schema.

---

## Common GPU Configurations

### AWS

| GPU | Instance Type | Accelerator Type |
|-----|---------------|------------------|
| NVIDIA A10G | `g5.xlarge` - `g5.48xlarge` | `nvidia-a10g` |
| NVIDIA A100 | `p4d.24xlarge` | `nvidia-a100` |
| NVIDIA H100 | `p5.48xlarge` | `nvidia-h100` |
| NVIDIA T4 | `g4dn.xlarge` - `g4dn.16xlarge` | `nvidia-tesla-t4` |

### GCP

| GPU | Instance Type | Accelerator Type |
|-----|---------------|------------------|
| NVIDIA L4 | `g2-standard-*` | `nvidia-l4` |
| NVIDIA A100 | `a2-highgpu-*` | `nvidia-a100` |
| NVIDIA H100 | `a3-highgpu-*` | `nvidia-h100` |
| NVIDIA T4 | `n1-standard-*` + GPU | `nvidia-tesla-t4` |

---

## Best Practices

1. **Separate GPU and CPU pools**: Create distinct node groups for GPU and CPU workloads
2. **Use SPOT for cost savings**: Non-critical workloads can use spot instances
3. **Set appropriate min/max**: Configure autoscaling limits based on expected demand
4. **Auxiliary node groups**: Use for system workloads to keep GPU nodes available for inference

---

## Related Documentation

- [Import a Cluster](./import-cluster.md)
- [Deploy to Your Cluster](../deployments/byoc-deployments.md)
- [Autoscaling Configuration](../platform-features/autoscaling.md)
