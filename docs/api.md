---
sidebar_position: 3
title: API Reference
---

# API Reference

```
Base URL: https://api.keyway.sh/v1
```

## Authentication

```bash
Authorization: Bearer <token>
```

| Token Type | Source | Use Case |
|------------|--------|----------|
| Keyway JWT | `keyway login` → `~/.config/keyway/config.json` | CLI, scripts |
| Keyway API Key | Dashboard → API Keys | CI/CD, automation |
| GitHub PAT | [Fine-grained PAT](https://github.com/settings/tokens?type=beta) | Legacy CI/CD |

---

## API Keys

Create and manage API keys from the [Dashboard](https://keyway.sh/dashboard/api-keys).

### Token Format

```
kw_live_a1B2c3D4e5F6g7H8i9J0k1L2m3N4o5P6q7R8s9T0
└─┘ └──┘ └────────────────────────────────────────┘
 │    │                    │
 │    │                    └── 40 chars base62 (240-bit entropy)
 │    └── Environment: live | test
 └── Prefix
```

### Scopes

| Scope | Permissions |
|-------|-------------|
| `read:secrets` | Pull secrets, list vaults |
| `write:secrets` | Push secrets, create/update vaults |
| `delete:secrets` | Delete secrets, trash vaults |
| `admin:api-keys` | Create/revoke API keys |

### Create API Key

```http
POST /v1/api-keys
{
  "name": "CI/CD Production",
  "environment": "live",
  "scopes": ["read:secrets", "write:secrets"],
  "expiresInDays": 365
}
```

Returns the full token **once**. Store it securely.

```json
{
  "data": {
    "id": "uuid",
    "name": "CI/CD Production",
    "token": "kw_live_a1B2c3D4...",
    "prefix": "kw_live_a1B2c3D4",
    "environment": "live",
    "scopes": ["read:secrets", "write:secrets"],
    "expiresAt": "2026-01-01T00:00:00Z"
  }
}
```

### List API Keys

```http
GET /v1/api-keys
```

Returns all keys (token is never returned after creation).

### Revoke API Key

```http
DELETE /v1/api-keys/:id
```

Immediate revocation. Cannot be undone.

### Usage Example

```bash
# Pull secrets with API key
curl -H "Authorization: Bearer kw_live_xxx" \
  "https://api.keyway.sh/v1/secrets/pull?repo=owner/repo&environment=production"
```

---

## Vaults

### List vaults

```http
GET /v1/vaults?limit=25&offset=0
```

### Create vault

```http
POST /v1/vaults
{ "repoFullName": "owner/repo" }
```

Requires admin access. Returns `409` if vault exists.

### Get vault

```http
GET /v1/vaults/:owner/:repo
```

### Delete vault

```http
DELETE /v1/vaults/:owner/:repo
```

Requires admin access. Deletes all secrets.

---

## Secrets

### List secrets (metadata only)

```http
GET /v1/vaults/:owner/:repo/secrets?limit=25&offset=0
```

### Create secret

```http
POST /v1/vaults/:owner/:repo/secrets
{
  "key": "DATABASE_URL",
  "value": "postgres://...",
  "environment": "production"
}
```

### Update secret

```http
PATCH /v1/vaults/:owner/:repo/secrets/:secretId
{ "value": "new_value" }
```

### Delete secret

```http
DELETE /v1/vaults/:owner/:repo/secrets/:secretId
```

### Get secret value

```http
GET /v1/vaults/:owner/:repo/secrets/:secretId/value
```

Returns `{ "value": "...", "preview": "post••••2/db" }`. Rate limited (10/min).

### View single secret (MCP/scripts)

```http
GET /v1/secrets/view?repo=owner/repo&environment=local&key=DATABASE_URL
```

Returns single secret value. Logged in audit trail.

---

## Bulk Operations (CLI)

### Push secrets

```http
POST /v1/secrets/push
{
  "repoFullName": "owner/repo",
  "environment": "local",
  "secrets": {
    "DATABASE_URL": "postgres://...",
    "API_KEY": "sk_test_..."
  }
}
```

Full sync: secrets not in payload are **deleted**.

### Pull secrets

```http
GET /v1/secrets/pull?repo=owner/repo&environment=local
```

Returns `.env` format in `data.content`.

---

## Environments

### List environments

```http
GET /v1/vaults/:owner/:repo/environments
```

### Create environment

```http
POST /v1/vaults/:owner/:repo/environments
{ "name": "preview" }
```

Requires admin. Name: 2-30 chars, lowercase, letters/numbers/dashes.

### Rename environment

```http
PATCH /v1/vaults/:owner/:repo/environments/:name
{ "newName": "development" }
```

### Delete environment

```http
DELETE /v1/vaults/:owner/:repo/environments/:name
```

Deletes all secrets in the environment.

---

## Users

### Get current user

```http
GET /v1/users/me
```

### Get usage

```http
GET /v1/users/me/usage
```

```json
{
  "data": {
    "plan": "free",
    "limits": { "maxPrivateRepos": 1 },
    "usage": { "private": 0, "public": 2 }
  }
}
```

---

## OAuth Device Flow

### 1. Start

```http
POST /v1/auth/device/start
{ "repository": "owner/repo" }
```

### 2. Poll

```http
POST /v1/auth/device/poll
{ "deviceCode": "abc123..." }
```

Returns `status`: `pending`, `approved`, `expired`, `denied`.

---

## Response Format

### Success

```json
{
  "data": { ... },
  "meta": { "requestId": "550e8400-..." }
}
```

### Error (RFC 7807)

```json
{
  "type": "https://keyway.sh/errors/not-found",
  "title": "Not Found",
  "status": 404,
  "detail": "Vault not found for repository owner/repo"
}
```

---

## Error Codes

| Status | Type | Description |
|--------|------|-------------|
| 400 | `bad-request` | Invalid request |
| 401 | `unauthorized` | Auth missing/invalid |
| 403 | `forbidden` | Insufficient permissions |
| 403 | `plan-limit` | Plan limit exceeded (includes `upgradeUrl`) |
| 404 | `not-found` | Resource not found |
| 409 | `conflict` | Resource already exists |
| 422 | `validation-error` | Invalid field values |
| 429 | `rate-limited` | Too many requests |

---

## Rate Limits

| Endpoint | Limit |
|----------|-------|
| General | 100 req / 15 min |
| Device verify | 5 req / min |
| Secret value | 10 req / min |

Headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `Retry-After`

---

## Validation Rules

### Secret keys

- Pattern: `^[A-Z][A-Z0-9_]{0,255}$`
- Max: 256 chars
- Valid: `DATABASE_URL`, `API_KEY_V2`

### Environment names

- Pattern: `^[a-z][a-z0-9_-]{1,29}$`
- 2-30 chars
- Valid: `local`, `dev-01`, `staging_v2`

### Secret values

- Max: 64 KB (256 KB on Team plan)
