# Chapter 27: Bash and Docker

## Introduction

Bash and Docker work together powerfully for container management and automation. This chapter covers Docker automation, container management, and deployment scripts.

---

## Docker Basics with Bash

### Running Containers

```bash
# Run container
docker run -d --name myapp nginx

# Run with environment variables
docker run -d --name myapp \
    -e DB_HOST=localhost \
    -e DB_PORT=3306 \
    nginx

# Run with volume
docker run -d --name myapp \
    -v /host/path:/container/path \
    nginx

# Run with port mapping
docker run -d --name myapp \
    -p 8080:80 \
    nginx
```

### Container Management Script

```bash
#!/bin/bash

CONTAINER_NAME="myapp"
IMAGE="nginx:latest"

start_container() {
    if docker ps -a | grep -q "$CONTAINER_NAME"; then
        echo "Container exists, starting..."
        docker start "$CONTAINER_NAME"
    else
        echo "Creating and starting container..."
        docker run -d \
            --name "$CONTAINER_NAME" \
            -p 80:80 \
            "$IMAGE"
    fi
}

stop_container() {
    docker stop "$CONTAINER_NAME"
}

remove_container() {
    docker rm -f "$CONTAINER_NAME"
}

case "${1:-}" in
    start) start_container ;;
    stop) stop_container ;;
    remove) remove_container ;;
    restart)
        stop_container
        start_container
        ;;
    *)
        echo "Usage: $0 {start|stop|remove|restart}"
        exit 1
        ;;
esac
```

---

## Docker Cleanup Automation

### Clean Unused Resources

```bash
#!/bin/bash
# docker-cleanup.sh

set -euo pipefail

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*"
}

log "Starting Docker cleanup..."

# Remove stopped containers
log "Removing stopped containers..."
docker container prune -f

# Remove unused images
log "Removing unused images..."
docker image prune -a -f

# Remove unused volumes
log "Removing unused volumes..."
docker volume prune -f

# Remove unused networks
log "Removing unused networks..."
docker network prune -f

# Show disk usage
log "Current disk usage:"
docker system df

log "Cleanup completed"
```

### Cleanup with Age Filter

```bash
#!/bin/bash

# Remove containers stopped more than 7 days ago
docker ps -a --filter "status=exited" --format "{{.ID}} {{.CreatedAt}}" | \
while read container_id created_at; do
    created_timestamp=$(date -d "$created_at" +%s)
    current_timestamp=$(date +%s)
    age_days=$(( ($current_timestamp - $created_timestamp) / 86400 ))
    
    if [ $age_days -gt 7 ]; then
        echo "Removing container $container_id (${age_days} days old)"
        docker rm "$container_id"
    fi
done
```

---

## Log Management

### View and Parse Docker Logs

```bash
#!/bin/bash

CONTAINER=$1

# Tail logs
docker logs -f "$CONTAINER"

# Get last 100 lines
docker logs --tail 100 "$CONTAINER"

# Logs since timestamp
docker logs --since "2025-01-01T00:00:00" "$CONTAINER"

# Logs with timestamps
docker logs -t "$CONTAINER"
```

### Log Analysis Script

```bash
#!/bin/bash
# analyze-docker-logs.sh

CONTAINER=$1
HOURS=${2:-1}

echo "=== Analyzing logs for $CONTAINER (last $HOURS hours) ==="

# Get logs
logs=$(docker logs --since "${HOURS}h" "$CONTAINER" 2>&1)

# Count errors
error_count=$(echo "$logs" | grep -ci "error")
echo "Errors: $error_count"

# Count warnings
warning_count=$(echo "$logs" | grep -ci "warning")
echo "Warnings: $warning_count"

# Show recent errors
if [ $error_count -gt 0 ]; then
    echo -e "\nRecent Errors:"
    echo "$logs" | grep -i "error" | tail -10
fi

# Check for common issues
if echo "$logs" | grep -qi "out of memory"; then
    echo "ALERT: Out of memory errors detected!"
fi

if echo "$logs" | grep -qi "connection refused"; then
    echo "ALERT: Connection refused errors detected!"
fi
```

---

## Docker Stats Monitoring

### Monitor Container Resources

```bash
#!/bin/bash
# monitor-containers.sh

THRESHOLD_CPU=80
THRESHOLD_MEM=80

# Get container stats
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemPerc}}" | \
tail -n +2 | \
while read name cpu mem; do
    cpu_val=$(echo "$cpu" | sed 's/%//')
    mem_val=$(echo "$mem" | sed 's/%//')
    
    echo "Container: $name"
    echo "  CPU: $cpu"
    echo "  Memory: $mem"
    
    if (( $(echo "$cpu_val > $THRESHOLD_CPU" | bc -l) )); then
        echo "  ALERT: High CPU usage!"
    fi
    
    if (( $(echo "$mem_val > $THRESHOLD_MEM" | bc -l) )); then
        echo "  ALERT: High memory usage!"
    fi
    
    echo "---"
done
```

---

## Docker Compose Automation

### Manage Docker Compose Services

```bash
#!/bin/bash
# docker-compose-manager.sh

set -euo pipefail

COMPOSE_FILE="docker-compose.yml"
ENVIRONMENT=${1:-production}

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*"
}

deploy() {
    log "Deploying to $ENVIRONMENT..."
    
    # Pull latest images
    docker-compose -f "$COMPOSE_FILE" pull
    
    # Stop and remove old containers
    docker-compose -f "$COMPOSE_FILE" down
    
    # Start new containers
    docker-compose -f "$COMPOSE_FILE" up -d
    
    # Wait for services to be healthy
    sleep 10
    
    # Check status
    docker-compose -f "$COMPOSE_FILE" ps
    
    log "Deployment completed"
}

rollback() {
    log "Rolling back..."
    
    # Use previous image tags
    docker-compose -f "$COMPOSE_FILE" down
    git checkout HEAD~1 -- "$COMPOSE_FILE"
    docker-compose -f "$COMPOSE_FILE" up -d
    
    log "Rollback completed"
}

health_check() {
    docker-compose -f "$COMPOSE_FILE" ps | grep "Up" > /dev/null
    if [ $? -eq 0 ]; then
        log "Health check: PASSED"
    else
        log "Health check: FAILED"
        return 1
    fi
}

case "${2:-deploy}" in
    deploy) deploy ;;
    rollback) rollback ;;
    health) health_check ;;
    *)
        echo "Usage: $0 <environment> {deploy|rollback|health}"
        exit 1
        ;;
esac
```

---

## Build and Push Scripts

### Multi-Architecture Build

```bash
#!/bin/bash
# multi-arch-build.sh

set -euo pipefail

IMAGE_NAME="myapp"
IMAGE_TAG=${1:-latest}
REGISTRY="docker.io/myorg"
PLATFORMS="linux/amd64,linux/arm64,linux/arm/v7"

# Setup buildx
docker buildx create --use --name multiarch || docker buildx use multiarch

# Build and push for multiple platforms
docker buildx build \
    --platform "$PLATFORMS" \
    --tag "$REGISTRY/$IMAGE_NAME:$IMAGE_TAG" \
    --tag "$REGISTRY/$IMAGE_NAME:latest" \
    --push \
    .

echo "Multi-architecture build completed"
```

---

## Best Practices

1. **Use health checks** in Dockerfiles
2. **Implement proper logging**
3. **Clean up regularly**
4. **Monitor resource usage**
5. **Use multi-stage builds**
6. **Version your images**
7. **Scan for vulnerabilities**

---

## Summary

- Automate container lifecycle management
- Clean up unused resources regularly
- Monitor container logs and stats
- Manage Docker Compose deployments
- Build multi-architecture images

---

**Next Chapter:** [28 - Bash and Kubernetes](28-bash-kubernetes.md)
