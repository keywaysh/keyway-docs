---
sidebar_position: 5
title: CI/CD Integration
---

# CI/CD Integration

Keyway integrates seamlessly with CI/CD pipelines. This guide covers common platforms.

## Authentication in CI/CD

Use a **GitHub Fine-grained Personal Access Token** for CI/CD:

1. Go to [GitHub Settings → Tokens](https://github.com/settings/tokens?type=beta)
2. Generate new token (fine-grained)
3. Select repository access
4. Required permissions: `Contents: Read`
5. Store the token as a CI secret

## GitHub Actions

### Using the Keyway Action (Recommended)

The easiest way to use Keyway in GitHub Actions:

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

      - name: Deploy (secrets are now in env)
        run: ./deploy.sh
```

The action automatically:
- Pulls secrets from your Keyway vault
- Exports them as environment variables
- Masks values in workflow logs

### Quick setup with CLI

Set up the `KEYWAY_TOKEN` secret automatically:

```bash
keyway ci setup
```

This command detects your repo and adds the secret for you (uses `gh` CLI if available).

### Action inputs

| Input | Description | Default |
|-------|-------------|---------|
| `token` | Keyway authentication token | Required |
| `environment` | Vault environment | `production` |
| `export-env` | Export as env vars | `true` |
| `env-file` | Write to .env file | - |
| `mask-values` | Mask values in logs | `true` |

### Write to .env file

```yaml
- uses: keywaysh/keyway-action@v1
  with:
    token: ${{ secrets.KEYWAY_TOKEN }}
    env-file: .env
    export-env: false
```

### Multiple environments

```yaml
jobs:
  test:
    steps:
      - uses: keywaysh/keyway-action@v1
        with:
          token: ${{ secrets.KEYWAY_TOKEN }}
          environment: development

  deploy:
    steps:
      - uses: keywaysh/keyway-action@v1
        with:
          token: ${{ secrets.KEYWAY_TOKEN }}
          environment: production
```

### Alternative: Using the CLI

If you prefer using the CLI directly:

```yaml
- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: '20'

- name: Install Keyway CLI
  run: npm install -g @keywaysh/cli

- name: Pull secrets
  run: keyway pull -e production -y
  env:
    KEYWAY_TOKEN: ${{ secrets.KEYWAY_TOKEN }}
```

## GitLab CI

```yaml
stages:
  - deploy

deploy:
  stage: deploy
  image: node:20
  before_script:
    - npm install -g @keywaysh/cli
  script:
    - keyway pull -e production -y
    - ./deploy.sh
  variables:
    KEYWAY_TOKEN: $KEYWAY_PAT
```

Store `KEYWAY_PAT` in GitLab CI/CD Variables (Settings → CI/CD → Variables).

## CircleCI

```yaml
version: 2.1

jobs:
  deploy:
    docker:
      - image: cimg/node:20.0
    steps:
      - checkout
      - run:
          name: Install Keyway CLI
          command: npm install -g @keywaysh/cli
      - run:
          name: Pull secrets
          command: keyway pull -e production -y
          environment:
            KEYWAY_TOKEN: ${KEYWAY_PAT}
      - run:
          name: Deploy
          command: ./deploy.sh

workflows:
  deploy:
    jobs:
      - deploy
```

Store `KEYWAY_PAT` in CircleCI Environment Variables.

## Jenkins

```groovy
pipeline {
    agent any

    environment {
        KEYWAY_TOKEN = credentials('keyway-pat')
    }

    stages {
        stage('Setup') {
            steps {
                sh 'npm install -g @keywaysh/cli'
            }
        }

        stage('Pull Secrets') {
            steps {
                sh 'keyway pull -e production -y'
            }
        }

        stage('Deploy') {
            steps {
                sh './deploy.sh'
            }
        }
    }
}
```

## Docker builds

### Build-time secrets

For Docker builds that need secrets:

```dockerfile
# Dockerfile
FROM node:20-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci

COPY . .

# Secrets are pulled at build time
ARG DATABASE_URL
ENV DATABASE_URL=$DATABASE_URL

RUN npm run build
```

```yaml
# GitHub Actions
- name: Pull secrets
  run: keyway pull -e production -y -f .env.build

- name: Build Docker image
  run: |
    source .env.build
    docker build \
      --build-arg DATABASE_URL=$DATABASE_URL \
      -t myapp:latest .
```

### Runtime secrets

Better approach - pull secrets at runtime:

```dockerfile
FROM node:20-alpine

WORKDIR /app
RUN npm install -g @keywaysh/cli

COPY package*.json ./
RUN npm ci
COPY . .

# Pull secrets at container start
CMD keyway pull -y && npm start
```

## Vercel

Vercel has built-in environment variables, but you can sync from Keyway:

```yaml
# GitHub Action to sync secrets to Vercel
- name: Pull from Keyway
  run: keyway pull -e production -y

- name: Sync to Vercel
  run: |
    while IFS='=' read -r key value; do
      vercel env add "$key" production <<< "$value"
    done < .env
  env:
    VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}
```

## Best practices

### 1. Use environment-specific tokens

Create separate PATs for different environments:
- `KEYWAY_PAT_STAGING` - access to staging only
- `KEYWAY_PAT_PRODUCTION` - access to production

### 2. Minimize secret exposure

Pull secrets only when needed:

```yaml
# Good - secrets only in deploy job
jobs:
  test:
    # No secrets needed

  deploy:
    needs: test
    steps:
      - run: keyway pull -e production -y
```

### 3. Don't log secrets

Ensure your CI doesn't log secret values:

```yaml
- name: Pull secrets
  run: keyway pull -e production -y > /dev/null
```

### 4. Rotate CI tokens

Set calendar reminders to rotate CI/CD tokens periodically.

### 5. Use read-only access

CI/CD typically only needs to read secrets, not write them. Use read-only repository access for the PAT.
