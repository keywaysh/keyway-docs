---
sidebar_position: 4
title: CI/CD
---

# CI/CD Integration

## Authentication

Use a [GitHub Fine-grained PAT](https://github.com/settings/tokens?type=beta) with `Contents: Read` permission.

---

## GitHub Actions

### Keyway Action (Recommended)

```yaml
name: Deploy
on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: keywaysh/keyway-action@v1
        with:
          token: ${{ secrets.KEYWAY_TOKEN }}
          environment: production

      - run: ./deploy.sh  # Secrets are in env vars
```

The action pulls secrets, exports them as env vars, and masks values in logs.

### Action Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `token` | required | Keyway token |
| `environment` | `production` | Vault environment |
| `export-env` | `true` | Export as env vars |
| `env-file` | - | Write to .env file |
| `mask-values` | `true` | Mask in logs |

### Quick Setup

```bash
keyway ci setup  # Adds KEYWAY_TOKEN secret to repo
```

### Write to File

```yaml
- uses: keywaysh/keyway-action@v1
  with:
    token: ${{ secrets.KEYWAY_TOKEN }}
    env-file: .env
    export-env: false
```

### Using CLI Directly

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: '20'

- run: npm install -g @keywaysh/cli

- run: keyway pull -e production -y
  env:
    KEYWAY_TOKEN: ${{ secrets.KEYWAY_TOKEN }}
```

---

## GitLab CI

```yaml
deploy:
  image: node:20
  before_script:
    - npm install -g @keywaysh/cli
  script:
    - keyway pull -e production -y
    - ./deploy.sh
  variables:
    KEYWAY_TOKEN: $KEYWAY_PAT
```

Store `KEYWAY_PAT` in Settings → CI/CD → Variables.

---

## CircleCI

```yaml
version: 2.1

jobs:
  deploy:
    docker:
      - image: cimg/node:20.0
    steps:
      - checkout
      - run: npm install -g @keywaysh/cli
      - run:
          command: keyway pull -e production -y
          environment:
            KEYWAY_TOKEN: ${KEYWAY_PAT}
      - run: ./deploy.sh
```

---

## Jenkins

```groovy
pipeline {
    agent any
    environment {
        KEYWAY_TOKEN = credentials('keyway-pat')
    }
    stages {
        stage('Setup') {
            steps { sh 'npm install -g @keywaysh/cli' }
        }
        stage('Pull') {
            steps { sh 'keyway pull -e production -y' }
        }
        stage('Deploy') {
            steps { sh './deploy.sh' }
        }
    }
}
```

---

## Docker

### Runtime (Recommended)

```dockerfile
FROM node:20-alpine
WORKDIR /app
RUN npm install -g @keywaysh/cli
COPY . .
CMD keyway pull -y && npm start
```

### Build-time

```yaml
- run: keyway pull -e production -y -f .env.build

- run: |
    source .env.build
    docker build --build-arg DATABASE_URL=$DATABASE_URL -t app .
```

---

## Best Practices

1. **Environment-specific tokens** - Separate PATs for staging/production
2. **Pull only when needed** - Not in test jobs
3. **Don't log secrets** - Use `> /dev/null` if needed
4. **Read-only access** - CI only needs read permissions
5. **Rotate tokens** - Periodically refresh CI tokens
