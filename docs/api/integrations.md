---
sidebar_position: 7
title: Integrations
---

# Integrations API

Connect and sync secrets with external providers like Vercel.

## List available providers

Returns the list of supported integration providers.

```http
GET /v1/integrations
```

**Response:**

```json
{
  "providers": [
    {
      "id": "vercel",
      "name": "Vercel",
      "description": "Deploy and sync environment variables with Vercel"
    }
  ]
}
```

---

## List connections

Returns the user's connected providers.

```http
GET /v1/integrations/connections
Authorization: Bearer <token>
```

**Response:**

```json
{
  "connections": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "provider": "vercel",
      "providerAccountId": "team_xxx",
      "createdAt": "2025-01-15T10:00:00Z"
    }
  ]
}
```

---

## Connect to a provider

Starts the OAuth flow to connect a provider. Redirects to the provider's authorization page.

```http
GET /v1/integrations/:provider/authorize
Authorization: Bearer <token>
```

**Query parameters:**

| Parameter | Description |
|-----------|-------------|
| `redirect_uri` | Optional. URL to redirect after successful connection |

**Example:**

```bash
# Opens browser to Vercel authorization
curl -L "https://api.keyway.sh/v1/integrations/vercel/authorize" \
  -H "Authorization: Bearer $TOKEN"
```

---

## Delete connection

Removes a provider connection.

```http
DELETE /v1/integrations/connections/:id
Authorization: Bearer <token>
```

**Response:**

```json
{
  "success": true
}
```

---

## List projects

Returns the projects available for a connection.

```http
GET /v1/integrations/connections/:id/projects
Authorization: Bearer <token>
```

**Response:**

```json
{
  "projects": [
    {
      "id": "prj_xxx",
      "name": "my-app",
      "framework": "nextjs"
    }
  ]
}
```

---

## Get sync status

Check if secrets are in sync between Keyway and a provider project.

```http
GET /v1/vaults/:owner/:repo/sync/status
Authorization: Bearer <token>
```

**Query parameters:**

| Parameter | Default | Description |
|-----------|---------|-------------|
| `connectionId` | required | Provider connection ID |
| `projectId` | required | Provider project ID |
| `environment` | `production` | Keyway environment to check |

**Response:**

```json
{
  "inSync": false,
  "lastSyncAt": "2025-01-15T10:00:00Z",
  "keywaySecretCount": 12,
  "providerSecretCount": 10
}
```

---

## Preview sync

Preview what changes would be made during a sync operation.

```http
GET /v1/vaults/:owner/:repo/sync/preview
Authorization: Bearer <token>
```

**Query parameters:**

| Parameter | Default | Description |
|-----------|---------|-------------|
| `connectionId` | required | Provider connection ID |
| `projectId` | required | Provider project ID |
| `keywayEnvironment` | `production` | Keyway environment |
| `providerEnvironment` | `production` | Provider environment |
| `direction` | `push` | `push` (Keyway → Provider) or `pull` (Provider → Keyway) |
| `allowDelete` | `false` | Include deletions in preview |

**Response:**

```json
{
  "changes": {
    "create": ["NEW_API_KEY", "NEW_SECRET"],
    "update": ["DATABASE_URL"],
    "delete": [],
    "unchanged": ["EXISTING_KEY"]
  },
  "stats": {
    "create": 2,
    "update": 1,
    "delete": 0,
    "unchanged": 1
  }
}
```

---

## Execute sync

Sync secrets between Keyway and a provider.

```http
POST /v1/vaults/:owner/:repo/sync
Authorization: Bearer <token>
Content-Type: application/json
```

**Request body:**

```json
{
  "connectionId": "550e8400-e29b-41d4-a716-446655440000",
  "projectId": "prj_xxx",
  "keywayEnvironment": "production",
  "providerEnvironment": "production",
  "direction": "push",
  "allowDelete": false
}
```

| Field | Default | Description |
|-------|---------|-------------|
| `connectionId` | required | Provider connection ID |
| `projectId` | required | Provider project ID |
| `keywayEnvironment` | `production` | Keyway environment |
| `providerEnvironment` | `production` | Provider environment |
| `direction` | `push` | `push` (Keyway → Provider) or `pull` (Provider → Keyway) |
| `allowDelete` | `false` | Delete secrets not present in source |

**Response:**

```json
{
  "success": true,
  "stats": {
    "created": 2,
    "updated": 1,
    "deleted": 0,
    "skipped": 0,
    "total": 3
  }
}
```

---

## Supported providers

| Provider | Status | Features |
|----------|--------|----------|
| Vercel | Available | Push/pull secrets, multiple environments |
| Netlify | Coming soon | - |
| Railway | Coming soon | - |
| Fly.io | Coming soon | - |
