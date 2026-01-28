# Autoscaling

Simplismart provides intelligent autoscaling to automatically adjust your deployment's replica count based on resource utilization.

---

## Overview

Autoscaling ensures your deployments:

- **Handle traffic spikes**: Scale up automatically when demand increases
- **Optimize costs**: Scale down (including to zero) when idle
- **Maintain performance**: Keep resource utilization within target thresholds

---

## Configuration

Autoscaling is configured via the `autoscale_config` field when creating a deployment.

### Basic Structure

```json
{
  "autoscale_config": {
    "targets": [
      {
        "metric": "gpu",
        "target": 80
      }
    ]
  }
}
```

### Supported Metrics

| Metric | Description | Best For |
|--------|-------------|----------|
| `cpu` | Average CPU utilization (%) | CPU-bound workloads |
| `gpu` | Average GPU utilization (%) | GPU inference workloads |
| `ram` | Average memory utilization (%) | Memory-intensive workloads |
| `gram` | Average GPU memory utilization (%) | Large model inference |
| `latency` | Average latency (ms) | Latency-sensitive workloads |
| `throughput` | Average throughput (RPS) | Throughput-sensitive workloads |
| `concurrency` | Average concurrency (requests per pod) | Concurrency-sensitive workloads |
| `queue_length` | Average queue length (active messages per pod) for async deployments | Async workloads |

---

## Updating Autoscaling Configuration and Replicas

Modify replica bounds for an existing deployment:

**Endpoint:** `POST /deployments/{deployment_id}/update-replicas/`

```bash
curl -X POST "https://api.app.simplismart.ai/deployments/{deployment_id}/update-replicas/" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "min_replicas": 2,
    "max_replicas": 10
  }'
```

See [Deployment Lifecycle](../deployments/lifecycle.md#update-replica-count) for full API documentation.

---

## Fast Scaleup

For latency-sensitive workloads, enable fast scaleup:

```json
{
  "fast_scaleup": true
}
```

### Benefits

- Pre-warmed container images (warmpool for AWS and Azure clusters **created** via Simplismart, **not** for self-managed imported clusters)
- Reduced cold start latency
- Faster response to traffic spikes

### Requirements

- Feature flag must be enabled for your organization
- Additional costs may apply

---

## Troubleshooting

### Deployment Not Scaling Up

1. **Check quota**: Verify sufficient GPU/CPU quota
2. **Check nodes**: Ensure node groups can accommodate more pods
3. **Check metrics**: Verify utilization metrics are being collected

### Deployment Not Scaling Down

1. **Check cooldown**: Wait for the cooldown period
2. **Check traffic**: Ensure traffic has actually decreased
3. **Check min_replicas**: Verify minimum isn't preventing scale-down

### Flapping (Frequent Scale Up/Down)

1. **Increase target**: Higher target reduces sensitivity
2. **Smooth traffic**: Consider request queuing
3. **Adjust min/max**: Reduce the scaling range

### Cold Start Issues

1. **Increase min_replicas**: Keep at least 1 pod warm
2. **Enable fast_scaleup**: Pre-warm containers

---

## Quotas

Autoscaling is subject to your organization's resource quotas, applicable only for private deployments, not BYOC:

- GPU quota limits maximum GPU pods
- CPU quota limits maximum CPU pods
- View quotas in the Simplismart settings page

Exceeding quotas results in:

```json
{
  "error": "Insufficient GPU quota for requested replicas"
}
```

---

## Related Documentation

- [BYOC Deployments](../deployments/byoc-deployments.md)
- [Private Deployments](../deployments/private-deployments.md)
- [Deployment Lifecycle](../deployments/lifecycle.md)
- [Monitoring](./monitoring.md)
