# Chapter 26: Bash for DevOps Pipelines

## Introduction

Bash is essential for DevOps automation. This chapter covers using Bash in CI/CD pipelines, with Jenkins, GitHub Actions, and other DevOps tools.

---

## CI/CD Pipeline Basics

### What is CI/CD?

**Continuous Integration (CI)**: Automatically build and test code changes
**Continuous Deployment (CD)**: Automatically deploy to production

### Bash's Role in Pipelines

- Build automation
- Testing and validation
- Deployment scripts
- Environment setup
- Reporting and notifications

---

## Jenkins Pipeline Scripts

### Basic Jenkinsfile with Bash

```groovy
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                sh '''
                    echo "Building application..."
                    npm install
                    npm run build
                '''
            }
        }
        
        stage('Test') {
            steps {
                sh '''
                    echo "Running tests..."
                    npm test
                '''
            }
        }
        
        stage('Deploy') {
            steps {
                sh '''
                    echo "Deploying application..."
                    ./deploy.sh production
                '''
            }
        }
    }
}
```

### Jenkins Deployment Script

```bash
#!/bin/bash
# deploy.sh

set -euo pipefail

ENVIRONMENT=${1:-staging}
APP_NAME="myapp"
DEPLOY_DIR="/var/www/$APP_NAME"
BACKUP_DIR="/var/backups/$APP_NAME"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*"
}

log "Starting deployment to $ENVIRONMENT"

# Backup current version
if [ -d "$DEPLOY_DIR" ]; then
    log "Backing up current version..."
    tar -czf "$BACKUP_DIR/backup_$(date +%Y%m%d_%H%M%S).tar.gz" "$DEPLOY_DIR"
fi

# Deploy new version
log "Deploying new version..."
rsync -avz --delete dist/ "$DEPLOY_DIR/"

# Set permissions
chown -R www-data:www-data "$DEPLOY_DIR"
chmod -R 755 "$DEPLOY_DIR"

# Restart services
log "Restarting services..."
systemctl restart nginx
systemctl restart "$APP_NAME"

log "Deployment completed successfully"
```

---

## GitHub Actions with Bash

### Basic GitHub Actions Workflow

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Setup
      run: |
        chmod +x ./scripts/*.sh
        ./scripts/setup.sh
    
    - name: Build
      run: |
        ./scripts/build.sh
    
    - name: Test
      run: |
        ./scripts/test.sh
    
    - name: Deploy
      if: github.ref == 'refs/heads/main'
      env:
        DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
      run: |
        ./scripts/deploy.sh production
```

### Build Script for GitHub Actions

```bash
#!/bin/bash
# scripts/build.sh

set -euo pipefail

echo "::group::Building application"

# Install dependencies
npm ci

# Run build
npm run build

# Check build artifacts
if [ ! -d "dist" ]; then
    echo "::error::Build failed - dist directory not found"
    exit 1
fi

echo "::endgroup::"
echo "Build completed successfully"
```

### Test Script with Reporting

```bash
#!/bin/bash
# scripts/test.sh

set -euo pipefail

echo "::group::Running tests"

# Run tests with coverage
npm test -- --coverage

# Check coverage threshold
coverage=$(jq -r '.total.lines.pct' coverage/coverage-summary.json)

if (( $(echo "$coverage < 80" | bc -l) )); then
    echo "::warning::Code coverage is below 80%: ${coverage}%"
else
    echo "::notice::Code coverage: ${coverage}%"
fi

echo "::endgroup::"
```

---

## GitLab CI with Bash

### GitLab CI Configuration

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - deploy

variables:
  APP_NAME: "myapp"

before_script:
  - chmod +x ./scripts/*.sh

build:
  stage: build
  script:
    - ./scripts/build.sh
  artifacts:
    paths:
      - dist/
    expire_in: 1 day

test:
  stage: test
  script:
    - ./scripts/test.sh
  coverage: '/Coverage: \d+\.\d+%/'

deploy_staging:
  stage: deploy
  script:
    - ./scripts/deploy.sh staging
  only:
    - develop
  environment:
    name: staging

deploy_production:
  stage: deploy
  script:
    - ./scripts/deploy.sh production
  only:
    - main
  when: manual
  environment:
    name: production
```

---

## Terraform Automation

### Terraform Wrapper Script

```bash
#!/bin/bash
# terraform-deploy.sh

set -euo pipefail

ENVIRONMENT=$1
TF_DIR="terraform/$ENVIRONMENT"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*"
}

error_exit() {
    log "ERROR: $1"
    exit 1
}

# Validate parameters
if [ -z "$ENVIRONMENT" ]; then
    error_exit "Environment not specified"
fi

if [ ! -d "$TF_DIR" ]; then
    error_exit "Terraform directory not found: $TF_DIR"
fi

cd "$TF_DIR"

# Initialize
log "Initializing Terraform..."
terraform init -input=false || error_exit "Terraform init failed"

# Validate
log "Validating configuration..."
terraform validate || error_exit "Terraform validation failed"

# Plan
log "Creating plan..."
terraform plan -out=tfplan -input=false || error_exit "Terraform plan failed"

# Apply
log "Applying changes..."
terraform apply -input=false tfplan || error_exit "Terraform apply failed"

# Cleanup
rm -f tfplan

log "Terraform deployment completed successfully"
```

### Terraform State Backup

```bash
#!/bin/bash
# backup-terraform-state.sh

ENVIRONMENTS=("dev" "staging" "production")
BACKUP_DIR="/backup/terraform"
S3_BUCKET="s3://terraform-states-backup"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

for env in "${ENVIRONMENTS[@]}"; do
    echo "Backing up $env state..."
    
    # Pull current state
    cd "terraform/$env"
    terraform state pull > "$BACKUP_DIR/${env}_${DATE}.tfstate"
    
    # Upload to S3
    aws s3 cp "$BACKUP_DIR/${env}_${DATE}.tfstate" \
        "$S3_BUCKET/$env/" \
        --storage-class GLACIER
done

# Cleanup old local backups (keep last 7 days)
find "$BACKUP_DIR" -name "*.tfstate" -mtime +7 -delete

echo "Terraform state backup completed"
```

---

## Ansible Automation

### Ansible Playbook Wrapper

```bash
#!/bin/bash
# run-ansible.sh

set -euo pipefail

PLAYBOOK=$1
ENVIRONMENT=${2:-staging}
INVENTORY="inventories/$ENVIRONMENT/hosts.yml"

# Validate inputs
if [ -z "$PLAYBOOK" ]; then
    echo "Usage: $0 <playbook> [environment]"
    exit 1
fi

if [ ! -f "$PLAYBOOK" ]; then
    echo "Playbook not found: $PLAYBOOK"
    exit 1
fi

# Ansible vault password
if [ -f ".vault_pass" ]; then
    VAULT_OPTS="--vault-password-file=.vault_pass"
else
    VAULT_OPTS=""
fi

# Run playbook
ansible-playbook \
    -i "$INVENTORY" \
    $VAULT_OPTS \
    --diff \
    "$PLAYBOOK"

echo "Ansible playbook completed: $PLAYBOOK"
```

---

## Docker Build Automation

### Docker Build Script

```bash
#!/bin/bash
# docker-build.sh

set -euo pipefail

IMAGE_NAME="myapp"
IMAGE_TAG=${1:-latest}
REGISTRY="docker.io/myorg"
DOCKERFILE="Dockerfile"

log() {
    echo "[BUILD] $*"
}

# Build image
log "Building Docker image: $IMAGE_NAME:$IMAGE_TAG"

docker build \
    -t "$IMAGE_NAME:$IMAGE_TAG" \
    -t "$IMAGE_NAME:latest" \
    -f "$DOCKERFILE" \
    .

# Tag for registry
docker tag "$IMAGE_NAME:$IMAGE_TAG" "$REGISTRY/$IMAGE_NAME:$IMAGE_TAG"
docker tag "$IMAGE_NAME:latest" "$REGISTRY/$IMAGE_NAME:latest"

# Push to registry
log "Pushing to registry..."
docker push "$REGISTRY/$IMAGE_NAME:$IMAGE_TAG"
docker push "$REGISTRY/$IMAGE_NAME:latest"

# Cleanup old images
log "Cleaning up old images..."
docker image prune -f

log "Docker build completed: $REGISTRY/$IMAGE_NAME:$IMAGE_TAG"
```

### Multi-Stage Build Script

```bash
#!/bin/bash
# multi-stage-build.sh

set -euo pipefail

SERVICES=("api" "web" "worker")
VERSION=${1:-$(git rev-parse --short HEAD)}
REGISTRY="docker.io/myorg"

build_service() {
    local service=$1
    
    echo "Building $service..."
    
    docker build \
        --target "$service" \
        -t "$REGISTRY/$service:$VERSION" \
        -t "$REGISTRY/$service:latest" \
        .
    
    docker push "$REGISTRY/$service:$VERSION"
    docker push "$REGISTRY/$service:latest"
}

# Build all services
for service in "${SERVICES[@]}"; do
    build_service "$service" &
done

# Wait for all builds
wait

echo "All services built successfully"
```

---

## Deployment Reports

### Generate Deployment Report

```bash
#!/bin/bash
# generate-deploy-report.sh

ENVIRONMENT=$1
VERSION=$2
REPORT_FILE="deploy_report_$(date +%Y%m%d_%H%M%S).md"

cat << EOF > "$REPORT_FILE"
# Deployment Report

**Environment:** $ENVIRONMENT
**Version:** $VERSION
**Date:** $(date +'%Y-%m-%d %H:%M:%S')
**Deployed By:** ${USER}

## Changes

\`\`\`
$(git log --oneline --no-merges -10)
\`\`\`

## Services Status

| Service | Status | Version |
|---------|--------|---------|
$(systemctl list-units --type=service --state=running | grep myapp | awk '{print "| " $1 " | Running | - |"}')

## Health Checks

$(curl -s https://$ENVIRONMENT.example.com/health | jq .)

## Rollback Command

\`\`\`bash
./deploy.sh $ENVIRONMENT $(git rev-parse --short HEAD~1)
\`\`\`

EOF

echo "Report generated: $REPORT_FILE"

# Upload to S3 or send via email
# aws s3 cp "$REPORT_FILE" s3://deployment-reports/
```

---

## Handling Secrets

### Safe Secret Management

```bash
#!/bin/bash
# manage-secrets.sh

set -euo pipefail

# Never hardcode secrets!
# Use environment variables or secret management tools

get_secret() {
    local secret_name=$1
    
    # AWS Secrets Manager
    aws secretsmanager get-secret-value \
        --secret-id "$secret_name" \
        --query SecretString \
        --output text
}

# Use secrets safely
DB_PASSWORD=$(get_secret "myapp/database/password")

# Use in deployment
export DB_PASSWORD
./deploy-with-secrets.sh

# Clear from environment
unset DB_PASSWORD
```

### Vault Integration

```bash
#!/bin/bash
# vault-secrets.sh

VAULT_ADDR="https://vault.example.com"
VAULT_TOKEN_FILE="/tmp/.vault-token"

# Login to Vault
vault login -method=github -token-only > "$VAULT_TOKEN_FILE"
export VAULT_TOKEN=$(cat "$VAULT_TOKEN_FILE")

# Read secrets
DB_USER=$(vault kv get -field=username secret/myapp/database)
DB_PASS=$(vault kv get -field=password secret/myapp/database)

# Use secrets in deployment
export DB_USER DB_PASS
./deploy.sh

# Cleanup
unset DB_USER DB_PASS VAULT_TOKEN
rm -f "$VAULT_TOKEN_FILE"
```

---

## Pipeline Health Monitoring

### Monitor Pipeline Status

```bash
#!/bin/bash
# monitor-pipelines.sh

JENKINS_URL="https://jenkins.example.com"
JENKINS_USER="admin"
JENKINS_TOKEN="$JENKINS_API_TOKEN"

check_job_status() {
    local job_name=$1
    
    status=$(curl -s -u "$JENKINS_USER:$JENKINS_TOKEN" \
        "$JENKINS_URL/job/$job_name/lastBuild/api/json" | \
        jq -r '.result')
    
    if [ "$status" = "SUCCESS" ]; then
        echo "[OK] $job_name: $status"
    else
        echo "[FAIL] $job_name: $status"
        # Send alert
    fi
}

JOBS=("build" "test" "deploy-staging" "deploy-production")

for job in "${JOBS[@]}"; do
    check_job_status "$job"
done
```

---

## Best Practices

1. **Use version control** for all scripts
2. **Implement proper error handling**
3. **Log all deployment actions**
4. **Never hardcode secrets**
5. **Make deployments idempotent**
6. **Test in staging first**
7. **Implement rollback procedures**
8. **Monitor pipeline health**

---

## Summary

- Integrate Bash with CI/CD tools (Jenkins, GitHub Actions, GitLab)
- Automate Terraform and Ansible workflows
- Build and deploy Docker containers
- Generate deployment reports
- Manage secrets securely
- Monitor pipeline health

---

**Next Chapter:** [27 - Bash and Docker](27-bash-docker.md)
