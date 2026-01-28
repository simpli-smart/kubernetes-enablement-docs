# Import Cluster API

Connect your existing Kubernetes cluster to Simplismart for Bring Your Own Compute (BYOC) deployments.

## Overview

When you import a cluster, Simplismart connects to your existing Kubernetes infrastructure, enabling you to:

- Deploy models and containers on your own hardware
- Maintain full control over your infrastructure
- Meet compliance and data residency requirements
- Leverage existing cloud investments

---

## Import a Cluster

Connect your existing Kubernetes cluster to Simplismart.

**Endpoint:** `POST /infrastructure/clusters/existing`

**Base URL:** `https://api.app.simplismart.ai`

**Authentication:** Bearer token
| Field | Type | Description |
|-------|------|-------------|
| `Authorization` | string | Bearer token |

**Example:**
```bash
curl -X POST "https://api.app.simplismart.ai/infrastructure/clusters/existing" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{ ... }'
```

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `org` | UUID | Your organization's UUID |
| `name` | string | A friendly name for your cluster |
| `slug` | string | URL-safe identifier (e.g., `my-prod-cluster`) |
| `hosted_zone` | UUID | The UUID of your hosted zone (see: [Hosted Zones](https://docs.simplismart.ai/model-suite/integrations/hosted-zone)) |
| `cluster_hosted_on` | string | Cloud provider (see supported providers below) |
| `region` | string | Required for AWS and GCP (e.g., `us-west-2`) else `null` |
| `secret_id` | UUID | Secret containing your kubeconfig (see: [Secrets](https://docs.simplismart.ai/model-suite/integrations/secrets#json-format-for-adding-secrets)) |
| `avatar` | object | Display image configuration |
| `nodegroup_labels` | array | At least one node group definition (key for the label must be `simplismart.ai/node-pool-name`, see [Import AWS Cluster Guide](https://docs.simplismart.ai/guides)) |

### Supported Cloud Providers

| Provider Value | Description |
|----------------|-------------|
| `aws` | Amazon Web Services |
| `gcp` | Google Cloud Platform |
| `azure` | Microsoft Azure |
| `oci` | Oracle Cloud Infrastructure |
| `kubernetes` | Generic Kubernetes |
| `e2e` | E2E Cloud |
| `shakti-cloud` | Shakti Cloud |

### Optional Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `service_type` | string | `LoadBalancer` | How services are exposed (`LoadBalancer` or `NodePort`) |
| `public_ip` | string | — | Required when using `NodePort` |
| `node_port_http` | integer | — | HTTP port (required for `NodePort`) |
| `node_port_https` | integer | — | HTTPS port (required for `NodePort`) |
| `node_config` | object | — | Default node configuration |
| `cluster_tools` | object | — | Tool groups and their catalog UUIDs |

### Avatar Configuration

```json
{
  "avatar": {
    "image_url": "https://ui-avatars.com/api/?name=Production",
    "font_color": "#FFFFFF",
    "background_color": "#2563EB"
  }
}
```

### Example Request

```bash
curl -X POST "https://api.app.simplismart.ai/infrastructure/clusters/exsisting" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "org": "54360212-2263-44c6-afaf-1333544854f7",
    "name": "Production Cluster",
    "slug": "prod-cluster",
    "hosted_zone": "b13c6a8a-6e9a-4bb9-8a0c-0db8c1a5a63f",
    "cluster_hosted_on": "aws",
    "region": "us-west-2",
    "secret_id": "9d1a0a51-9af6-4c6f-90ab-1a8b54e2b7b1",
    "avatar": {
      "image_url": "https://ui-avatars.com/api/?name=Production",
      "font_color": "#FFFFFF",
      "background_color": "#2563EB"
    },
    "nodegroup_labels": [
      {
        "name": "gpu-pool",
        "is_auxilliary": false,
        "node_config": {
          "machine_type": "g5.2xlarge",
          "capacity_type": "ON_DEMAND",
          "min_node_count": 0,
          "max_node_count": 3,
          "desired_count": 1,
          "accelerator_type": "nvidia-a10g",
          "accelerator_count": 1
        }
      }
    ]
  }'
```

### Success Response

**Status:** `201 Created`

```json
{
  "uuid": "1e99a854-26de-4c19-b6b6-68c7e03c39c3",
  "name": "Production Cluster",
  "status": "active",
  "cluster_hosted_on": "aws",
  "region": "us-west-2",
  "node_config": { ... },
  "kubeconfig": { ... }
}
```

### Error Responses

#### Missing Region (400 Bad Request)

```json
{
  "error": "Region is required"
}
```

**Solution:** Add the `region` field for AWS or GCP clusters.

#### Validation Failed (422 Unprocessable Entity)

```json
{
  "error": "Failed to import cluster: <validation errors>"
}
```

**Solution:** Check that all required fields are present and correctly formatted.

#### Server Error (500 Internal Server Error)

```json
{
  "error": "Failed to import cluster: <reason>"
}
```

**Solution:** Contact support if issue persists.

---

## Node Group Configuration

Node groups define the compute resources in your cluster. You can specify multiple node groups, but only one can be marked as auxiliary (`is_auxilliary: true`).

> **NOTE:**
>
> An auxilliary node group is used to deploy the deployment controller and other tooling components.
> It is not used to deploy models or containers.

### Basic Node Group

```json
{
  "name": "cpu-pool",
  "is_auxilliary": false
}
```

### GPU Node Group

```json
{
  "name": "gpu-pool",
  "is_auxilliary": false,
  "node_config": {
    "machine_type": "g5.2xlarge",
    "capacity_type": "ON_DEMAND",
    "min_node_count": 0,
    "max_node_count": 3,
    "desired_count": 1,
    "accelerator_type": "nvidia-a10g",
    "accelerator_count": 1,
    "vcpu": 4,
    "memory": 4
  }
}
```

### Node Config Fields

| Field | Type | Description |
|-------|------|-------------|
| `machine_type` | string | Instance type (e.g., `g5.2xlarge`, `n1-standard-4`) |
| `capacity_type` | string | `ON_DEMAND` or `SPOT` |
| `min_node_count` | integer | Minimum nodes in the group |
| `max_node_count` | integer | Maximum nodes in the group |
| `desired_count` | integer | Initial node count |
| `accelerator_type` | string | GPU type (e.g., `nvidia-a10g`, `nvidia-h100`) |
| `accelerator_count` | integer | Number of GPUs per node |
| `vcpu` | integer | vCPU count |
| `memory` | integer | Memory in GB |

---

## Cluster Tools

Specify pre-configured tools for your cluster by grouping them into categories:

```json
{
  "cluster_tools": {
    "TRAINING": ["uuid-of-training-tool-1", "uuid-of-training-tool-2"],
    "ASYNC TOOLS": ["uuid-of-async-tool-1"]
  }
}
```

See [Cluster Tools](../platform-features/cluster-tools.md) for available tool catalogs.

---

## Next Steps

- [Add Node Groups to a Cluster](./node-groups.md)
- [Create an Imported Container](../containers/imported-containers.md)
- [Deploy to Your Cluster](../deployments/byoc-deployments.md)
