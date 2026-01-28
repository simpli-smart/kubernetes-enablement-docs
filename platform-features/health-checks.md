# Health Checks & Reconciliation

Simplismart provides proactive health monitoring with automatic reconciliation to ensure your deployments remain healthy and available.

---

## Overview

The health check system:

1. **Monitors continuously**: Regular probes to verify deployment health
2. **Detects issues early**: Identifies problems before they impact users
3. **Reports status**: Clear, actionable health information via Alerts and UI

---

## Health Check Configuration

Configure health checks when importing cluster by installing SS Agent as a cluster tooling. See [Importing a Cluster](../clusters/import-cluster.md) for more details.

---

## Health Status API

Query the current health status of a deployment.

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

## Health Status Values

| Status | Description | Typical Causes |
|--------|-------------|----------------|
| `Healthy` | All systems operational | Normal operation |
| `Progressing` | Deployment is changing | Scaling, updating |
| `Initializing` | Starting up | New deployment, restart |
| `Degraded` | Partially unhealthy | Some pods failing probes |
| `Unhealthy` | Not functioning | All pods failing, misconfig |
| `SCALED_DOWN` | Intentionally paused | Manual stop, scale-to-zero |
| `Unknown` | Cannot determine | Connectivity issues |
| `Missing` | No health data | Metrics collection failure |

---

## Implementing Health Endpoints for Imported Containers

Your container must expose a health endpoint that Simplismart can probe. See [Importing a Container](../containers/imported-containers.md) for more details.

### Requirements

1. **HTTP endpoint**: Must respond to HTTP GET requests
2. **No authentication**: Endpoint should be accessible without auth
3. **Fast response**: Should respond within timeout period
4. **Accurate status**: Return appropriate HTTP status codes

### HTTP Status Codes

| Status Code | Meaning | Result |
|-------------|---------|--------|
| `200-399` | Healthy | Pod marked as ready |
| `400-599` | Unhealthy | Pod marked as not ready |

> **Note:** The same health check endpoint is used for both liveness and readiness probes.

### Example: Python (FastAPI)

```python
from fastapi import FastAPI
from fastapi.responses import JSONResponse

app = FastAPI()

@app.get("/health")
async def health():
    if model_is_ready():
        return {"status": "healthy"}
    return JSONResponse(
        status_code=503,
        content={"status": "unhealthy"}
    )
```

---

## Related Documentation

- [Monitoring](./monitoring.md)
- [Autoscaling](./autoscaling.md)
- [Deployment Lifecycle](../deployments/lifecycle.md)
- [BYOC Deployments](../deployments/byoc-deployments.md)
