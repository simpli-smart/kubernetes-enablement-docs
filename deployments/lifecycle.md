# Deployment Lifecycle API

Manage the lifecycle of your deployments: list, health status, scaling, start/stop, and deletion.

---

## List Deployments

Retrieve all deployments for your organization.

**Endpoint:** `GET /deployments/list/model/`

**Base URL:** `https://api.app.simplismart.ai`

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model_repo_id` | UUID | No | Filter deployments by container |

### Example: List All Deployments

```bash
curl -X GET "https://api.app.simplismart.ai/deployments/list/model/" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### Example: Filter by Container

```bash
curl -X GET "https://api.app.simplismart.ai/deployments/list/model/?model_repo_id=0caac439-5518-4569-a3db-a2d505a189f2" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### Success Response

**Status:** `200 OK`

```json
[
  {
    "deployment_id": "1e99a854-26de-4c19-b6b6-68c7e03c39c3",
    "deployment_name": "whisper-v3",
    "model_repo_id": "0caac439-5518-4569-a3db-a2d505a189f2",
    "model_repo_name": "whisper-v3-arabic-h100",
    "model_type": "custom",
    "accelerator_type": ["nvidia-tesla-t4"],
    "accelerator_count": 1,
    "status": "DEPLOYED"
  },
  {
    "deployment_id": "3bda38dc-e4be-43c2-a5eb-4d23ed4d3970",
    "deployment_name": "llama-inference",
    "model_repo_id": "0caac439-5518-4569-a3db-a2d505a189f2",
    "model_repo_name": "llama-7b-optimized",
    "model_type": "custom",
    "accelerator_type": ["nvidia-l4"],
    "accelerator_count": 1,
    "status": "DEPLOYED"
  }
]
```

### Deployment Statuses

| Status | Description |
|--------|-------------|
| `DEPLOYED` | Deployment is active and healthy |
| `DEPLOYING` | Deployment is being created |
| `FAILED` | Deployment creation failed |
| `DELETED` | Deployment has been deleted |
| `DELETING` | Deployment is being deleted |
| `SCALED_DOWN` | Deployment is paused (stopped) or scaled to zero |

---

## Get Deployment Health

Check the current health status of a deployment.

**Endpoint:** `GET /deployments/model-deployment/fetch-status/{deployment_id}/`

**Base URL:** `https://api.app.simplismart.ai`

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `deployment_id` | UUID | Deployment UUID |

### Example Request

```bash
curl -X GET "https://api.app.simplismart.ai/deployments/model-deployment/fetch-status/1e99a854-26de-4c19-b6b6-68c7e03c39c3/" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### Success Response

**Status:** `200 OK`

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

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `data` | string | Aggregated health status |
| `messages` | array | Human-readable status messages with severity |
| `pods` | object | Pod readiness counts (optional) |

### Health Status Values

| Status | Description |
|--------|-------------|
| `Healthy` | Ready to serve traffic |
| `Progressing` | Starting or updating |
| `Initializing` | Being created |
| `Degraded` | Partially unhealthy |
| `Unhealthy` | Not functioning correctly |
| `Unknown` | Status could not be determined |
| `SCALED_DOWN` | Deployment is paused or scaled to zero |
| `Missing` | Health data unavailable |

### Example: Scaled Down Deployment

```json
{
  "data": "SCALED_DOWN",
  "messages": [
    {
      "message": "Deployment is paused. Resume to start inferencing.",
      "severity": "info"
    }
  ]
}
```

### Error Responses

| Status | Description |
|--------|-------------|
| `400 Bad Request` | Missing deployment ID or deployment is deleted |
| `404 Not Found` | Deployment does not exist |
| `500 Internal Server Error` | Unexpected failure |

---

## Update Replica Count

Adjust the autoscaling bounds for a deployment.

**Endpoint:** `POST /deployments/{deployment_id}/update-replicas/`

**Base URL:** `https://api.app.simplismart.ai`

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `deployment_id` | UUID | Deployment UUID |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `min_replicas` | integer | Yes | Minimum pods to keep running (≥ 0) |
| `max_replicas` | integer | Yes | Maximum pods allowed (≥ 1) |

### Validation Rules

- `min_replicas` must be ≥ 0
- `max_replicas` must be ≥ 1
- `min_replicas` must be ≤ `max_replicas`
- Deployment must be active (not FAILED, DELETING, or DELETED)
- Sufficient quota must be available

### Example Request

```bash
curl -X POST "https://api.app.simplismart.ai/deployments/1e99a854-26de-4c19-b6b6-68c7e03c39c3/update-replicas/" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "min_replicas": 2,
    "max_replicas": 4
  }'
```

### Success Response

**Status:** `200 OK`

```json
{
  "message": "Autoscaling configuration updated successfully",
  "min_replicas": 2,
  "max_replicas": 4
}
```

### Error Responses

#### Insufficient Quota (400 Bad Request)

```json
{
  "error": "Insufficient GPU quota for requested replicas"
}
```

#### Deployment Not Found (404 Not Found)

```json
{
  "error": "No active deployment found"
}
```

### Notes

- Changes are applied immediately
- Scaling may take time based on image size and model startup time
- Enable **fast scaleup** during deployment for faster scaling
- Quotas are visible in the Simplismart settings page

---

## Start / Stop Deployment

Pause or resume a deployment.

**Endpoint:** `POST /deployments/{deployment_id}/startstop/`

**Base URL:** `https://api.app.simplismart.ai`

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `deployment_id` | UUID | Deployment UUID |

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `action` | string | Yes | `start` or `stop` |
| `org` | UUID | Yes | Your organization UUID |

### Example: Stop a Deployment

```bash
curl -X POST "https://api.app.simplismart.ai/deployments/1e99a854-26de-4c19-b6b6-68c7e03c39c3/startstop/" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "action": "stop",
    "org": "54360212-2263-44c6-afaf-1333544854f7"
  }'
```

### Example: Start a Deployment

```bash
curl -X POST "https://api.app.simplismart.ai/deployments/1e99a854-26de-4c19-b6b6-68c7e03c39c3/startstop/" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "action": "start",
    "org": "54360212-2263-44c6-afaf-1333544854f7"
  }'
```

### Success Response

**Status:** `200 OK`

```json
{
  "message": "Deployment started successfully",
  "update_result": "started"
}
```

### Error Responses

| Status | Error | Description |
|--------|-------|-------------|
| `400 Bad Request` | Invalid action / missing org | Check request body |
| `402 Payment Required` | Wallet is blocked | Resolve billing issues |
| `500 Internal Server Error` | Failed to start/stop | Contact support |

---

## Delete Deployment

Permanently remove a deployment.

**Endpoint:** `DELETE /deployments/{deployment_id}/delete/`

**Base URL:** `https://api.app.simplismart.ai`

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `deployment_id` | UUID | Deployment UUID |

### Example Request

```bash
curl -X DELETE "https://api.app.simplismart.ai/deployments/1e99a854-26de-4c19-b6b6-68c7e03c39c3/delete/" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### Success Responses

#### Deployment Deleted

**Status:** `200 OK`

```json
{
  "message": "Deployment deleted successfully"
}
```

#### Deployment Already Deleted

**Status:** `200 OK`

```json
{
  "message": "Deployment already Deleted"
}
```

### Error Responses

#### Deployment Not Found (404 Not Found)

```json
{
  "error": "Deployment not found"
}
```

#### Multiple Deletion Failures (400 Bad Request)

```json
{
  "error": "Deletion failed multiple times already. Please contact support."
}
```

### Notes

- Deletion is **idempotent** — repeated deletes are safe
- Maximum 3 failed deletion attempts are allowed
- Some deletions may be asynchronous (warm pool deployments)
- After deletion, active usage tracking is ended

---

## Related Documentation

- [BYOC Deployments](./byoc-deployments.md)
- [Private Deployments](./private-deployments.md)
- [Health Checks](../platform-features/health-checks.md)
- [Autoscaling](../platform-features/autoscaling.md)
