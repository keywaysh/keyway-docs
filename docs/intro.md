---
slug: /
sidebar_position: 1
title: Getting Started
---

# Keyway

**GitHub-native secrets management.** If you have access to a repo, you have access to its secrets.

## Quick Start

**1. Initialize**

```bash
cd your-project
npx @keywaysh/cli init    # Opens browser for GitHub auth + syncs your .env
```

**2. Pull (on another machine or teammate)**

```bash
npx @keywaysh/cli pull
```

**3. Sync with a provider (optional)**

```bash
npx @keywaysh/cli sync vercel
```

That's it. Your team members with repo access can immediately pull secrets.

## How It Works

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   CLI       │────▶│   Keyway    │────▶│   GitHub    │
│  (your PC)  │     │    API      │     │    API      │
└─────────────┘     └─────────────┘     └─────────────┘
                           │
                    ┌──────┴──────┐
                    ▼             ▼
             ┌───────────┐ ┌─────────────┐
             │  Crypto   │ │  PostgreSQL │
             │ (isolated)│ │ (encrypted) │
             └───────────┘ └─────────────┘
```

- **CLI** authenticates via GitHub OAuth
- **API** verifies repo access via GitHub API
- **Secrets** encrypted with AES-256-GCM

## Team Access

GitHub repo permissions = Keyway permissions. No separate invitations.

**Personal repos:** Owner has full access, collaborators get read/write.

| Role | Can Read | Can Write |
|------|:--------:|:---------:|
| Owner | ✓ | ✓ |
| Collaborator | ✓ | ✓ |

**Organization repos:** Fine-grained roles available.

| Role | Can Read | Can Write | Can Admin |
|------|:--------:|:---------:|:---------:|
| Admin | ✓ | ✓ | ✓ |
| Maintain | ✓ | ✓ | - |
| Write | ✓ | ✓ | - |
| Triage | ✓ | - | - |
| Read | ✓ | - | - |

**Onboarding a teammate:**
1. Add them to GitHub repo
2. They run `keyway pull`

## Organizations

For teams, install the [Keyway GitHub App](https://github.com/apps/keyway-sh) on your organization to unlock:

- Centralized billing (Team plan for the whole org)
- Member sync from GitHub
- 14-day free trial
- Permission overrides per environment

See [Organizations](/organizations) for details.

## Environments

Default environments: `local`, `development`, `staging`, `production`

```bash
keyway push -e production
keyway pull -e staging
```

## Plans

| | Free | Pro ($9/mo) | Team ($29/mo) |
|--|:--:|:--:|:--:|
| Public repos | Unlimited | Unlimited | Unlimited |
| Private repos | 1 | Unlimited | Unlimited |
| Private org repos | - | - | Unlimited |
| Providers | 2 | Unlimited | Unlimited |
| Environments | 2 | Unlimited | Unlimited |
| Secret versions | - | 30 days | 30 days |
| Activity logs | 7 days | 30 days | 90 days |
| Permission overrides | - | - | ✓ |

Upgrade: [keyway.sh/settings](https://keyway.sh/settings)

## Next Steps

- [CLI Reference](/cli) - All commands
- [CI/CD](/ci-cd) - GitHub Actions, GitLab, and more
- [API](/api) - REST API and API keys
- [Organizations](/organizations) - Team billing and management
- [Security](/security) - Permissions and activity logs
