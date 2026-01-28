# Imported Containers API

Create and manage imported containers (your own Docker images) for deployment on Simplismart.

## Overview

Imported Containers allow you to bring your own Docker images from:

- **Docker Hub** - Private repositories
- **Depot** - Private repositories

Once imported, containers can be deployed to either Simplismart's managed cloud (Private Deployment) or your own clusters (BYOC).

---

## Create an Imported Container

Register a Docker image as a container in Simplismart.

**Endpoint:** `POST /models/client/model-repos/`

**Base URL:** `https://api.app.simplismart.ai`

**Authentication:** Bearer token
| Field | Type | Description |
|-------|------|-------------|
| `Authorization` | string | Bearer token |

**Example:**
```bash
curl -X POST "https://api.app.simplismart.ai/models/client/model-repos/" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{ ... }'
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Container name |
| `runtime_gpus` | integer | Yes | Number of GPUs required. Set to `0` for CPU-only deployments |
| `docker_tag` | string | Yes | Docker image tag (e.g., `latest`, `v1.0.0`) |
| `registry_path` | string | Yes | Docker image path (e.g., `myorg/myimage`) |
| `source_secret` | UUID | Yes | Your Docker registry secret UUID, see [Secrets](https://docs.simplismart.ai/model-suite/integrations/secrets) |
| `env` | object | No | Environment variables (default key-value pairs, can be overridden in deployment) |

### Example Request

```bash
curl -X POST "https://api.app.simplismart.ai/models/client/model-repos/" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "my-inference-server",
    "runtime_gpus": 1,
    "docker_tag": "latest",
    "registry_path": "myorg/inference-server",
    "source_secret": "9731422f-b437-4aa5-b33b-0398s3126e4e",
    "env": {
      "MODEL_PATH": "/models/llama-7b",
      "MAX_BATCH_SIZE": "32"
    }
  }'
```

### Success Response

**Status:** `201 Created`

```json
{
  "uuid": "a843be1e-f59a-415e-8b36-e6731dead1a0",
  "name": "my-inference-server",
  "source_type": "docker_hub",
  "source_url": "myorg/inference-server:latest",
  "is_byom": true,
  "accelerator": null,
  "runtime_gpus": 1,
  "byom": {
    "image": "myorg/inference-server:latest",
    "registry": "myorg/inference-server",
    "tag": "latest"
  },
  "secrets": {
    "source_secret": {
      "uuid": "9731422f-b437-4aa5-b33b-0398s3126e4e",
      "name": "docker-registry-secret"
    }
  },
  "status": "SUCCESS",
  "status_description": null,
  "model_type": "byom",
  "env": {
    "MODEL_PATH": "/models/llama-7b",
    "MAX_BATCH_SIZE": "32"
  },
  "created_at": "2025-12-26T09:40:33.418430Z",
  "updated_at": "2025-12-26T09:40:33.418453Z",
  "org_id": "54360212-2263-44c6-afaf-1333544854f7"
}
```

### Error Responses

#### Missing Required Field (400 Bad Request)

```json
{
  "detail": {
    "runtime_gpus": ["This field may not be null."]
  }
}
```

#### Invalid Docker Image (400 Bad Request)

```json
{
  "detail": {
    "non_field_errors": [
      "Could not validate Docker Image: myorg/invalid-image:latest against the provided secrets"
    ]
  }
}
```

**Solution:** Verify the image exists and the provided secret has access to the registry.

#### Server Error (500 Internal Server Error)

```json
{
  "detail": "An unexpected error occurred: <reason>"
}
```

---

## List Imported Containers

Retrieve all imported containers for your organization.

**Endpoint:** `GET /models/client/model-repos/`

**Base URL:** `https://api.app.simplismart.ai`

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `offset` | integer | No | Pagination offset (default: `0`) |
| `count` | integer | No | Number of items to return (default: `5`, max: `100`) |

### Example Request

```bash
curl -X GET "https://api.app.simplismart.ai/models/client/model-repos/?offset=0&count=10" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### Success Response

**Status:** `200 OK`

```json
[
  {
    "uuid": "18bf746d-2894-495f-a437-dab70667d36f",
    "name": "my-inference-server",
    "source_type": "docker_hub",
    "source_url": "myorg/inference-server:latest",
    "is_byom": true,
    "accelerator": null,
    "runtime_gpus": 1,
    "byom": {
      "image": "myorg/inference-server:latest",
      "registry": "myorg/inference-server",
      "tag": "latest"
    },
    "secrets": {
      "source_secret": {
        "uuid": "9731422f-b437-4aa5-b33b-0398s3126e4e",
        "name": "docker-registry-secret"
      }
    },
    "status": "SUCCESS",
    "status_description": null,
    "model_type": "byom",
    "env": {
      "MODEL_PATH": "/models/llama-7b"
    },
    "created_at": "2025-12-26T09:44:30.865639Z",
    "updated_at": "2025-12-26T09:44:30.865673Z",
    "org_id": "54360212-2263-44c6-afaf-1333544854f7"
  }
]
```

---

## Get a Single Container

Retrieve details for a specific imported container.

**Endpoint:** `GET /models/client/model-repos/?model_id={container_id}`

### Example Request

```bash
curl -X GET "https://api.app.simplismart.ai/models/client/model-repos/?model_id=a843be1e-f59a-415e-8b36-e6731dead1a0" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### Success Response

**Status:** `200 OK`

Returns a single container object (same structure as list response).

### Error Response

#### Container Not Found (404 Not Found)

```json
{
  "detail": "Model not found."
}
```

---

## Delete an Imported Container

Remove an imported container from Simplismart.

**Endpoint:** `DELETE /models/client/model-repos/?model_id={container_id}`

### Prerequisites

Before deleting a container, you must:
1. Delete all active deployments using this container
2. Ensure the container is not part of any pending operations

### Example Request

```bash
curl -X DELETE "https://api.app.simplismart.ai/models/client/model-repos/?model_id=a843be1e-f59a-415e-8b36-e6731dead1a0" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

### Success Response

**Status:** `200 OK`

```json
{
  "detail": "Model deleted successfully."
}
```

### Error Responses

#### Missing Container ID (400 Bad Request)

```json
{
  "detail": "model_id query param is required."
}
```

#### Already Deleted (400 Bad Request)

```json
{
  "detail": "Model is already deleted."
}
```

#### Active Deployments Exist (400 Bad Request)

```json
{
  "detail": "This model has active deployments. Delete deployments before deleting the model."
}
```

#### Container Not Found (404 Not Found)

```json
{
  "detail": "Model not found."
}
```

---

## Container Statuses

| Status | Description |
|--------|-------------|
| `SUCCESS` | Container is ready for deployment |
| `PENDING` | Container is being validated |
| `FAILED` | Container validation failed |

---

## Setting Up Registry Secrets

To import containers from private registries, you need to create a registry secret first. Visit the [Secrets Management](https://docs.simplismart.ai/model-suite/integrations/secrets) documentation for instructions on creating Docker registry secrets.

---

## Best Practices

1. **Use specific tags**: Avoid `latest` in production; use versioned tags for reproducibility
2. **Validate locally first**: Ensure your container runs correctly before importing
3. **Configure GPU requirements accurately**: Set `runtime_gpus` based on your actual requirements
4. **Use environment variables**: Configure runtime behavior through `env` rather than hardcoding in the container image

---

## Related Documentation

- [BYOC Deployments](../deployments/byoc-deployments.md)
- [Private Deployments](../deployments/private-deployments.md)
- [Secrets Management](https://docs.simplismart.ai/secrets)
