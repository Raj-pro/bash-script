# Chapter 30: CI/CD Environments

## Introduction

Managing environments and secrets in CI/CD pipelines is critical. This chapter covers environment configuration, secret management, and pipeline best practices.

---

## Environment Management

### Environment Configuration

```bash
#!/bin/bash
# load-environment.sh

ENVIRONMENT=${1:-development}
ENV_FILE="config/${ENVIRONMENT}.env"

if [ ! -f "$ENV_FILE" ]; then
    echo "Environment file not found: $ENV_FILE"
    exit 1
fi

# Load environment variables
set -a
source "$ENV_FILE"
set +a

echo "Loaded environment: $ENVIRONMENT"
echo "Database: $DB_HOST"
echo "API URL: $API_URL"
```

### Environment Switcher

```bash
#!/bin/bash
# switch-environment.sh

ENVIRONMENTS=("development" "staging" "production")
CURRENT_ENV=${1:-development}

validate_environment() {
    for env in "${ENVIRONMENTS[@]}"; do
        if [ "$env" = "$CURRENT_ENV" ]; then
            return 0
        fi
    done
    return 1
}

if ! validate_environment; then
    echo "Invalid environment: $CURRENT_ENV"
    echo "Valid options: ${ENVIRONMENTS[*]}"
    exit 1
fi

# Load environment-specific configuration
export NODE_ENV="$CURRENT_ENV"
export CONFIG_FILE="config/${CURRENT_ENV}.json"

# Deploy with environment
./deploy.sh "$CURRENT_ENV"
```

---

## Secret Management

### AWS Secrets Manager Integration

```bash
#!/bin/bash
# fetch-secrets.sh

get_secret() {
    local secret_name=$1
    
    aws secretsmanager get-secret-value \
        --secret-id "$secret_name" \
        --query SecretString \
        --output text
}

# Load secrets
export DB_PASSWORD=$(get_secret "myapp/db/password")
export API_KEY=$(get_secret "myapp/api/key")

# Run application
./start-app.sh

# Clean up
unset DB_PASSWORD API_KEY
```

### HashiCorp Vault Integration

```bash
#!/bin/bash
# vault-integration.sh

VAULT_ADDR="https://vault.example.com"
VAULT_NAMESPACE="myapp"

# Login to Vault
vault login -method=github

# Read secrets
DB_USER=$(vault kv get -field=username secret/$VAULT_NAMESPACE/database)
DB_PASS=$(vault kv get -field=password secret/$VAULT_NAMESPACE/database)

# Export for application
export DB_USER DB_PASS

# Run application
./app

# Cleanup
unset DB_USER DB_PASS
```

---

## Pipeline-Ready Scripts

### Pre-Deploy Validation

```bash
#!/bin/bash
# pre-deploy-check.sh

set -euo pipefail

ERRORS=0

check() {
    local name=$1
    shift
    
    if "$@"; then
        echo "[✓] $name"
    else
        echo "[✗] $name"
        ((ERRORS++))
    fi
}

echo "=== Pre-Deploy Validation ==="

# Check environment variables
check "Environment variables" [ -n "${DB_HOST:-}" ]
check "API URL configured" [ -n "${API_URL:-}" ]

# Check dependencies
check "Docker installed" command -v docker
check "kubectl installed" command -v kubectl

# Check connectivity
check "Database reachable" ping -c 1 "$DB_HOST"
check "API endpoint available" curl -sf "$API_URL/health"

# Check permissions
check "Can write to deploy dir" [ -w "/opt/deploy" ]

if [ $ERRORS -gt 0 ]; then
    echo "Pre-deploy validation failed with $ERRORS errors"
    exit 1
fi

echo "Pre-deploy validation passed"
```

---

## Deployment Strategies

### Canary Deployment

```bash
#!/bin/bash
# canary-deploy.sh

APP="myapp"
NEW_VERSION=$1
CANARY_PERCENTAGE=10

# Deploy canary
kubectl set image deployment/${APP}-canary \
    app=myorg/${APP}:${NEW_VERSION}

# Wait for canary
kubectl rollout status deployment/${APP}-canary

# Monitor metrics
sleep 60

# Check error rate
error_rate=$(curl -s "http://metrics/error_rate?app=${APP}-canary")

if (( $(echo "$error_rate > 1.0" | bc -l) )); then
    echo "Canary failed! Rolling back..."
    kubectl rollout undo deployment/${APP}-canary
    exit 1
fi

# Promote canary to production
kubectl set image deployment/${APP} \
    app=myorg/${APP}:${NEW_VERSION}

echo "Canary deployment successful"
```

---

## Best Practices

1. **Separate environments** clearly
2. **Never commit secrets** to git
3. **Use secret management** tools
4. **Validate before deploying**
5. **Implement rollback** procedures
6. **Log all deployments**
7. **Test in staging** first

---

## Summary

- Manage multiple environments
- Integrate secret management tools
- Validate deployments
- Implement deployment strategies
- Follow security best practices

---

**Next Chapter:** Part 9 - Professional Bash Scripting
