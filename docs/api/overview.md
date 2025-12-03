---
sidebar_position: 1
title: API Overview
---

# API Overview

The Keyway REST API allows you to programmatically manage vaults, secrets, and environments.

## Base URL

```
https://api.keyway.sh/v1
```

## Authentication

All API requests require authentication. Include your token in the `Authorization` header:

```bash
Authorization: Bearer <token>
```

### Token types

| Type | How to obtain | Use case |
|------|---------------|----------|
| **Keyway JWT** | `keyway login` then check `~/.config/keyway/config.json` | CLI, scripts |
| **GitHub PAT** | [Create a fine-grained PAT](https://github.com/settings/tokens?type=beta) | CI/CD, automation |

Both token types work interchangeably.

## Response format

All successful responses follow this structure:

```json
{
  "data": { ... },
  "meta": {
    "requestId": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

### Paginated responses

List endpoints include pagination metadata in the `meta` object:

```json
{
  "data": [...],
  "meta": {
    "requestId": "550e8400-e29b-41d4-a716-446655440000",
    "pagination": {
      "total": 100,
      "limit": 25,
      "offset": 0,
      "hasMore": true
    }
  }
}
```

Use `?limit=N&offset=M` query parameters to paginate.

### No Content responses

Some endpoints (like DELETE operations) return `204 No Content` with an empty body.

## Error format

Errors follow [RFC 7807](https://tools.ietf.org/html/rfc7807) Problem Details:

```json
{
  "type": "https://keyway.sh/errors/not-found",
  "title": "Not Found",
  "status": 404,
  "detail": "Vault not found for repository owner/repo",
  "instance": "/v1/vaults/owner/repo",
  "requestId": "550e8400-e29b-41d4-a716-446655440000"
}
```

### Common error codes

| Status | Type | Description |
|--------|------|-------------|
| 400 | `bad-request` | Invalid request body or parameters |
| 401 | `unauthorized` | Missing or invalid authentication |
| 403 | `forbidden` | Insufficient permissions |
| 404 | `not-found` | Resource not found |
| 409 | `conflict` | Resource already exists |
| 429 | `rate-limited` | Too many requests |

## Rate limits

| Endpoint | Limit |
|----------|-------|
| General | 100 requests / 15 minutes |
| Device verify | 5 requests / minute |

When rate limited, you'll receive a `429` response with a `Retry-After` header.

## API sections

- [Authentication](/api/authentication) - OAuth device flow, PAT validation
- [Vaults](/api/vaults) - Create, list, delete vaults
- [Secrets](/api/secrets) - Push, pull, manage secrets
- [Environments](/api/environments) - Manage vault environments
- [Integrations](/api/integrations) - Provider connections (Vercel, etc.)
- [Users](/api/users) - User profile and usage
