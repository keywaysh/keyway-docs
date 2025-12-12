---
sidebar_position: 6
title: Security
---

# Security & Permissions

## Encryption

| Layer | Method |
|-------|--------|
| At rest | AES-256-GCM (random IV per encryption) |
| In transit | TLS 1.3, HSTS |

---

## Permissions

GitHub repo access = Keyway vault access. No separate user management.

### Personal Repos

| Role | Read | Write | Admin |
|------|:----:|:-----:|:-----:|
| Owner | ✓ | ✓ | ✓ |
| Collaborator | ✓ | ✓ | - |

### Organization Repos

| Role | Read | Write | Admin |
|------|:----:|:-----:|:-----:|
| Admin | ✓ | ✓ | ✓ |
| Maintain | ✓ | ✓ | - |
| Write | ✓ | ✓ | - |
| Triage | ✓ | - | - |
| Read | ✓ | - | - |

### Operations by Permission

**Admin only:**
- Initialize vault (`keyway init`)
- Create/delete environments
- Delete vault

**Write (write, maintain, admin):**
- Push secrets
- Create/update/delete secrets
- Trigger provider syncs

**Read (all roles):**
- Pull secrets
- List secrets

---

## Authentication

### OAuth Device Flow (CLI)

1. CLI requests device code
2. User approves in browser
3. CLI receives token (30 days)

Token stored in `~/.config/keyway/config.json` (mode 600).

### GitHub PAT (CI/CD)

Use [fine-grained PATs](https://github.com/settings/tokens?type=beta) with `Contents: Read`.

---

## Team Management

**Add access:** Add user as GitHub collaborator → they run `keyway pull`

**Remove access:** Remove from GitHub repo → instant revocation

**Check access:**
```bash
keyway doctor
```

---

## Best Practices

1. **Never commit secrets** - `.gitignore` all `.env` files
2. **Least privilege** - Give minimum necessary access
3. **Rotate regularly** - Especially after team changes
4. **Separate environments** - Don't use production secrets locally
5. **Fine-grained PATs for CI** - Scope to specific repos

---

## Incident Response

**Compromised secret:**
1. Rotate immediately
2. `keyway push` new values
3. Deploy changes
4. Audit access

**Compromised token:**
1. `keyway logout` or revoke PAT
2. Re-authenticate
3. Review access logs

---

## Reporting Vulnerabilities

Email security@keyway.sh with reproduction steps. We respond within 24 hours.
