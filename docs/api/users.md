---
sidebar_position: 6
title: Users
---

# Users API

Get information about the authenticated user and their usage.

## Get current user

Returns the authenticated user's profile.

```http
GET /v1/users/me
Authorization: Bearer <token>
```

**Response:**

```json
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "username": "octocat",
    "githubId": 12345,
    "avatarUrl": "https://avatars.githubusercontent.com/u/12345",
    "createdAt": "2025-01-01T00:00:00Z"
  }
}
```

---

## Get usage statistics

Returns the user's current usage and plan limits.

```http
GET /v1/users/me/usage
Authorization: Bearer <token>
```

**Response:**

```json
{
  "data": {
    "plan": "free",
    "limits": {
      "maxPublicRepos": "unlimited",
      "maxPrivateRepos": 1
    },
    "usage": {
      "public": 2,
      "private": 0
    }
  }
}
```

### Usage limits by plan

| Plan | Public repos | Private repos |
|------|--------------|---------------|
| Free | Unlimited | 1 |
| Pro | Unlimited | Unlimited |
| Team | Unlimited | Unlimited |
