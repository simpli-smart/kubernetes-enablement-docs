# Authentication

All Simplismart API requests require authentication using a JWT (JSON Web Token).

## Obtaining Your Token

To obtain your JWT token, follow the instructions in the [Simplismart Authentication Guide](https://docs.simplismart.ai/model-suite/settings/api-keys).

## Using Your Token

Include your JWT token in the `Authorization` header of all API requests:

```
Authorization: Bearer YOUR_JWT_TOKEN
```

### Example Request

```bash
curl -X GET "https://api.app.simplismart.ai/deployments/list/model/" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json"
```

## Token Scoping

Your JWT token is scoped to your organization. This means:

- You can only access resources belonging to your organization
- Deployments, clusters, and containers are isolated per organization
- API responses are automatically filtered to your organization's data

## Security Best Practices

1. **Never share your token** - Treat your JWT like a password
2. **Don't commit tokens** - Use environment variables or secret management tools
3. **Rotate regularly** - Refresh tokens periodically for security
4. **Use HTTPS only** - All API requests must use HTTPS

## Environment Variables

We recommend storing your token in an environment variable:

```bash
export SIMPLISMART_TOKEN="your_jwt_token_here"
```

Then reference it in your API calls:

```bash
curl -X GET "https://api.app.simplismart.ai/deployments/list/model/" \
  -H "Authorization: Bearer $SIMPLISMART_TOKEN" \
  -H "Content-Type: application/json"
```

## Error Responses

### 401 Unauthorized

If your token is missing, expired, or invalid:

```json
{
  "detail": "Authentication credentials were not provided."
}
```

or

```json
{
  "detail": "Invalid token."
}
```

**Solution:** Verify your token is correct and hasn't expired. Obtain a new token if needed.

### 403 Forbidden

If your token doesn't have permission for the requested resource:

```json
{
  "detail": "You do not have permission to perform this action."
}
```

**Solution:** Ensure you're accessing resources within your organization and have the appropriate role (Admin or Member).

---

## Next Steps

- [Import a Cluster](../clusters/import-cluster.md)
- [Create an Imported Container](../containers/imported-containers.md)
- [Deploy to BYOC](../deployments/byoc-deployments.md)
