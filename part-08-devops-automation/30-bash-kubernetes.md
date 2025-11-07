# Chapter 28: Bash and Kubernetes

## Introduction

Kubernetes automation with Bash enables efficient cluster management and deployment workflows. This chapter covers kubectl automation and k8s scripting.

---

## kubectl Basics

### Common Operations

```bash
# Get resources
kubectl get pods
kubectl get services
kubectl get deployments

# Describe resource
kubectl describe pod my-pod

# View logs
kubectl logs my-pod

# Execute command in pod
kubectl exec -it my-pod -- /bin/bash

# Apply configuration
kubectl apply -f deployment.yaml

# Delete resource
kubectl delete pod my-pod
```

---

## Deployment Automation

### Deploy Application Script

```bash
#!/bin/bash
# k8s-deploy.sh

set -euo pipefail

APP_NAME="myapp"
NAMESPACE="production"
IMAGE_TAG=${1:-latest}
MANIFEST_DIR="k8s"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*"
}

log "Deploying $APP_NAME:$IMAGE_TAG to $NAMESPACE..."

# Update image tag
sed -i "s|image:.*|image: myorg/$APP_NAME:$IMAGE_TAG|" \
    "$MANIFEST_DIR/deployment.yaml"

# Apply manifests
kubectl apply -f "$MANIFEST_DIR/" -n "$NAMESPACE"

# Wait for rollout
kubectl rollout status deployment/$APP_NAME -n "$NAMESPACE"

log "Deployment completed successfully"
```

### Blue-Green Deployment

```bash
#!/bin/bash
# blue-green-deploy.sh

set -euo pipefail

APP="myapp"
NAMESPACE="production"
NEW_VERSION=$1

# Deploy green environment
kubectl apply -f k8s/deployment-green.yaml -n "$NAMESPACE"

# Wait for green to be ready
kubectl wait --for=condition=available \
    deployment/${APP}-green -n "$NAMESPACE" --timeout=300s

# Run health checks
if ! ./health-check.sh "${APP}-green"; then
    echo "Health check failed, rolling back..."
    kubectl delete deployment/${APP}-green -n "$NAMESPACE"
    exit 1
fi

# Switch traffic to green
kubectl patch service "$APP" -n "$NAMESPACE" \
    -p '{"spec":{"selector":{"version":"green"}}}'

# Remove blue environment
kubectl delete deployment/${APP}-blue -n "$NAMESPACE"

echo "Blue-green deployment completed"
```

---

## Scaling Automation

### Auto-Scale Pods

```bash
#!/bin/bash
# scale-pods.sh

APP=$1
REPLICAS=$2
NAMESPACE=${3:-default}

# Scale deployment
kubectl scale deployment/"$APP" \
    --replicas="$REPLICAS" \
    -n "$NAMESPACE"

# Wait for scaling
kubectl rollout status deployment/"$APP" -n "$NAMESPACE"

echo "Scaled $APP to $REPLICAS replicas"
```

### Load-Based Scaling

```bash
#!/bin/bash
# auto-scale-check.sh

APP="myapp"
NAMESPACE="production"
MAX_CPU=80
MIN_REPLICAS=2
MAX_REPLICAS=10

# Get current CPU usage
cpu_usage=$(kubectl top pods -n "$NAMESPACE" -l app="$APP" \
    | awk '{sum+=$3} END {print sum}' | sed 's/m//')

current_replicas=$(kubectl get deployment "$APP" -n "$NAMESPACE" \
    -o jsonpath='{.spec.replicas}')

echo "Current CPU: ${cpu_usage}m, Replicas: $current_replicas"

# Scale up if CPU high
if [ "$cpu_usage" -gt "$MAX_CPU" ] && [ "$current_replicas" -lt "$MAX_REPLICAS" ]; then
    new_replicas=$((current_replicas + 1))
    kubectl scale deployment/"$APP" --replicas="$new_replicas" -n "$NAMESPACE"
    echo "Scaled up to $new_replicas replicas"
fi
```

---

## Monitoring and Health Checks

### Cluster Health Check

```bash
#!/bin/bash
# k8s-health-check.sh

echo "=== Kubernetes Cluster Health ==="

# Node status
echo "Node Status:"
kubectl get nodes

# Check system pods
echo -e "\nSystem Pods:"
kubectl get pods -n kube-system

# Check for unhealthy pods
echo -e "\nUnhealthy Pods:"
kubectl get pods --all-namespaces --field-selector=status.phase!=Running,status.phase!=Succeeded

# Resource usage
echo -e "\nTop Nodes:"
kubectl top nodes

echo -e "\nTop Pods:"
kubectl top pods --all-namespaces | head -10
```

---

## Backup and Restore

### Backup Kubernetes Resources

```bash
#!/bin/bash
# k8s-backup.sh

BACKUP_DIR="/backup/k8s/$(date +%Y%m%d)"
NAMESPACES=$(kubectl get namespaces -o jsonpath='{.items[*].metadata.name}')

mkdir -p "$BACKUP_DIR"

for ns in $NAMESPACES; do
    echo "Backing up namespace: $ns"
    
    mkdir -p "$BACKUP_DIR/$ns"
    
    # Backup all resources
    for resource in deployment service configmap secret ingress; do
        kubectl get "$resource" -n "$ns" -o yaml \
            > "$BACKUP_DIR/$ns/${resource}.yaml" 2>/dev/null || true
    done
done

echo "Backup completed: $BACKUP_DIR"
```

---

## Best Practices

1. **Use namespaces** for isolation
2. **Implement health checks**
3. **Monitor resource usage**
4. **Automate rollbacks**
5. **Use ConfigMaps/Secrets** for configuration
6. **Test in staging first**

---

## Summary

- Automate kubectl operations
- Implement deployment strategies
- Scale applications dynamically
- Monitor cluster health
- Backup Kubernetes resources

---

**Next Chapter:** [29 - Cloud Integration (AWS, Azure, GCP)](29-cloud-integration.md)
