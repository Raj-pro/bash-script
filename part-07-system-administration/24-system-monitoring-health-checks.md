# Chapter 22: System Monitoring and Health Checks

## Introduction

System monitoring ensures your infrastructure runs smoothly. This chapter covers monitoring disk usage, memory, CPU, and automating health checks with Bash scripts.

---

## Disk Monitoring

### Check Disk Usage

```bash
# Overall disk usage
df -h

# Specific filesystem
df -h /home

# Inode usage
df -i

# Show filesystem type
df -hT
```

### Disk Usage Script

```bash
#!/bin/bash

THRESHOLD=80
EMAIL="admin@example.com"

check_disk_usage() {
    df -H | grep -vE '^Filesystem|tmpfs|cdrom' | awk '{print $5 " " $1}' | while read output; do
        usage=$(echo $output | awk '{print $1}' | cut -d'%' -f1)
        partition=$(echo $output | awk '{print $2}')
        
        if [ $usage -ge $THRESHOLD ]; then
            echo "WARNING: Disk $partition is ${usage}% full"
            # Send alert
            # mail -s "Disk Alert: $partition" $EMAIL <<< "Partition $partition is ${usage}% full"
        fi
    done
}

check_disk_usage
```

### Find Large Files

```bash
#!/bin/bash

# Find files larger than 100MB
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null

# Top 20 largest files
find / -type f -exec du -h {} + 2>/dev/null | sort -rh | head -n 20

# Largest directories
du -h --max-depth=1 / 2>/dev/null | sort -rh | head -n 20
```

### Disk Space Report

```bash
#!/bin/bash

generate_disk_report() {
    local report_file="disk_report_$(date +%Y%m%d).txt"
    
    {
        echo "=== Disk Usage Report ==="
        echo "Generated: $(date)"
        echo
        
        echo "Filesystem Usage:"
        df -h | grep -vE 'tmpfs|devtmpfs'
        echo
        
        echo "Top 10 Directories by Size:"
        du -h --max-depth=1 /home 2>/dev/null | sort -rh | head -10
        echo
        
        echo "Recently Modified Large Files (>50MB):"
        find /var/log -type f -size +50M -mtime -7 -exec ls -lh {} \; 2>/dev/null
        
    } > "$report_file"
    
    echo "Report generated: $report_file"
}

generate_disk_report
```

---

## Memory Monitoring

### Check Memory Usage

```bash
# Memory info
free -h

# Detailed memory
cat /proc/meminfo

# Per-process memory
ps aux --sort=-%mem | head -n 10
```

### Memory Monitoring Script

```bash
#!/bin/bash

THRESHOLD=90

check_memory() {
    # Get memory usage percentage
    memory_usage=$(free | grep Mem | awk '{print ($3/$2) * 100.0}')
    memory_usage_int=${memory_usage%.*}
    
    echo "Memory Usage: ${memory_usage_int}%"
    
    if [ "$memory_usage_int" -gt "$THRESHOLD" ]; then
        echo "WARNING: Memory usage is high!"
        
        # Show top memory consumers
        echo "Top 5 Memory Consumers:"
        ps aux --sort=-%mem | head -n 6
        
        # Send alert
        # mail -s "High Memory Alert" admin@example.com
    fi
}

check_memory
```

### Memory Details Script

```bash
#!/bin/bash

memory_details() {
    echo "=== Memory Statistics ==="
    echo
    
    echo "Overall Memory:"
    free -h
    echo
    
    echo "Top 10 Memory Consuming Processes:"
    ps aux --sort=-%mem | awk 'NR<=11{printf "%-10s %-8s %-8s %-s\n", $1, $2, $4"%", $11}'
    echo
    
    echo "Memory by Application:"
    ps aux | awk '{mem[$11]+=$ 6} END {for (app in mem) printf "%-30s %10.2f MB\n", app, mem[app]/1024}' | sort -k2 -rn | head -10
    echo
    
    echo "Swap Usage:"
    swapon --show
}

memory_details
```

---

## CPU Monitoring

### Check CPU Usage

```bash
# CPU info
lscpu

# Current CPU usage
top -bn1 | grep "Cpu(s)"

# Per-core usage
mpstat -P ALL

# Load average
uptime
```

### CPU Monitoring Script

```bash
#!/bin/bash

CPU_THRESHOLD=80

check_cpu() {
    # Get CPU usage
    cpu_usage=$(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print 100 - $1}')
    cpu_usage_int=${cpu_usage%.*}
    
    echo "CPU Usage: ${cpu_usage_int}%"
    
    if [ "$cpu_usage_int" -gt "$CPU_THRESHOLD" ]; then
        echo "WARNING: High CPU usage detected!"
        
        echo "Top CPU Consuming Processes:"
        ps aux --sort=-%cpu | head -n 6
        
        # Send alert
    fi
}

check_cpu
```

### Load Average Monitor

```bash
#!/bin/bash

check_load_average() {
    # Get number of CPU cores
    cpu_cores=$(nproc)
    
    # Get load averages
    load_1=$(uptime | awk -F'load average:' '{print $2}' | awk -F, '{print $1}' | xargs)
    load_5=$(uptime | awk -F'load average:' '{print $2}' | awk -F, '{print $2}' | xargs)
    load_15=$(uptime | awk -F'load average:' '{print $2}' | awk -F, '{print $3}' | xargs)
    
    echo "CPU Cores: $cpu_cores"
    echo "Load Average: $load_1 (1m), $load_5 (5m), $load_15 (15m)"
    
    # Check if load exceeds number of cores
    if (( $(echo "$load_1 > $cpu_cores" | bc -l) )); then
        echo "WARNING: 1-minute load average exceeds CPU cores!"
    fi
}

check_load_average
```

---

## Comprehensive Health Check

### All-in-One Health Check Script

```bash
#!/bin/bash

DISK_THRESHOLD=85
MEM_THRESHOLD=85
CPU_THRESHOLD=80
LOAD_THRESHOLD=2

RED='\033[0;31m'
YELLOW='\033[1;33m'
GREEN='\033[0;32m'
NC='\033[0m' # No Color

health_check() {
    echo "======================================"
    echo "      SYSTEM HEALTH CHECK"
    echo "      $(date)"
    echo "======================================"
    echo
    
    # Disk Check
    echo "--- DISK USAGE ---"
    while read line; do
        usage=$(echo "$line" | awk '{print $5}' | sed 's/%//')
        mount=$(echo "$line" | awk '{print $6}')
        
        if [ "$usage" -ge "$DISK_THRESHOLD" ]; then
            echo -e "${RED}[CRITICAL]${NC} $mount: ${usage}%"
        elif [ "$usage" -ge 70 ]; then
            echo -e "${YELLOW}[WARNING]${NC} $mount: ${usage}%"
        else
            echo -e "${GREEN}[OK]${NC} $mount: ${usage}%"
        fi
    done < <(df -h | grep -vE 'tmpfs|devtmpfs|Filesystem')
    echo
    
    # Memory Check
    echo "--- MEMORY USAGE ---"
    mem_usage=$(free | grep Mem | awk '{print ($3/$2) * 100.0}')
    mem_usage_int=${mem_usage%.*}
    
    if [ "$mem_usage_int" -ge "$MEM_THRESHOLD" ]; then
        echo -e "${RED}[CRITICAL]${NC} Memory: ${mem_usage_int}%"
    elif [ "$mem_usage_int" -ge 70 ]; then
        echo -e "${YELLOW}[WARNING]${NC} Memory: ${mem_usage_int}%"
    else
        echo -e "${GREEN}[OK]${NC} Memory: ${mem_usage_int}%"
    fi
    echo
    
    # CPU Check
    echo "--- CPU USAGE ---"
    cpu_usage=$(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print 100 - $1}')
    cpu_usage_int=${cpu_usage%.*}
    
    if [ "$cpu_usage_int" -ge "$CPU_THRESHOLD" ]; then
        echo -e "${RED}[CRITICAL]${NC} CPU: ${cpu_usage_int}%"
    elif [ "$cpu_usage_int" -ge 60 ]; then
        echo -e "${YELLOW}[WARNING]${NC} CPU: ${cpu_usage_int}%"
    else
        echo -e "${GREEN}[OK]${NC} CPU: ${cpu_usage_int}%"
    fi
    echo
    
    # Load Average
    echo "--- LOAD AVERAGE ---"
    cpu_cores=$(nproc)
    load_1=$(uptime | awk -F'load average:' '{print $2}' | awk -F, '{print $1}' | xargs)
    
    echo "CPU Cores: $cpu_cores"
    echo "Load (1m): $load_1"
    
    if (( $(echo "$load_1 > ($cpu_cores * $LOAD_THRESHOLD)" | bc -l) )); then
        echo -e "${RED}[CRITICAL]${NC} Load is high"
    elif (( $(echo "$load_1 > $cpu_cores" | bc -l) )); then
        echo -e "${YELLOW}[WARNING]${NC} Load approaching limit"
    else
        echo -e "${GREEN}[OK]${NC} Load normal"
    fi
    echo
    
    # Service Status
    echo "--- CRITICAL SERVICES ---"
    for service in sshd nginx mysql docker; do
        if systemctl is-active --quiet "$service" 2>/dev/null; then
            echo -e "${GREEN}[RUNNING]${NC} $service"
        else
            echo -e "${RED}[STOPPED]${NC} $service"
        fi
    done
    echo
    
    # Network
    echo "--- NETWORK ---"
    if ping -c 1 8.8.8.8 &>/dev/null; then
        echo -e "${GREEN}[OK]${NC} Internet connectivity"
    else
        echo -e "${RED}[FAIL]${NC} No internet connection"
    fi
    echo
    
    echo "======================================"
}

health_check
```

---

## Service Monitoring

### Check Service Status

```bash
#!/bin/bash

SERVICES=("nginx" "mysql" "redis" "sshd")

check_services() {
    echo "=== Service Status ==="
    
    for service in "${SERVICES[@]}"; do
        if systemctl is-active --quiet "$service"; then
            echo "[OK] $service is running"
        else
            echo "[FAIL] $service is NOT running"
            
            # Try to start the service
            echo "Attempting to start $service..."
            systemctl start "$service"
            
            sleep 2
            
            if systemctl is-active --quiet "$service"; then
                echo "[OK] $service started successfully"
            else
                echo "[FAIL] Could not start $service"
                # Send alert
            fi
        fi
    done
}

check_services
```

---

## Log Analysis

### Check for Errors in Logs

```bash
#!/bin/bash

check_logs() {
    local log_file=$1
    local hours=${2:-1}
    
    echo "=== Checking $log_file (last $hours hours) ==="
    
    # Errors
    error_count=$(find "$log_file" -mmin -$((hours * 60)) -exec grep -i "error" {} \; 2>/dev/null | wc -l)
    echo "Errors: $error_count"
    
    # Warnings
    warning_count=$(find "$log_file" -mmin -$((hours * 60)) -exec grep -i "warning" {} \; 2>/dev/null | wc -l)
    echo "Warnings: $warning_count"
    
    # Critical
    critical_count=$(find "$log_file" -mmin -$((hours * 60)) -exec grep -i "critical" {} \; 2>/dev/null | wc -l)
    echo "Critical: $critical_count"
    
    if [ "$error_count" -gt 100 ] || [ "$critical_count" -gt 0 ]; then
        echo "ALERT: High error rate detected!"
        # Show recent errors
        grep -i "error\|critical" "$log_file" | tail -n 10
    fi
}

# Check various logs
check_logs "/var/log/syslog" 1
check_logs "/var/log/nginx/error.log" 1
```

---

## Network Monitoring

### Network Connectivity Check

```bash
#!/bin/bash

HOSTS=("google.com" "github.com" "8.8.8.8")

check_connectivity() {
    echo "=== Network Connectivity ==="
    
    for host in "${HOSTS[@]}"; do
        if ping -c 1 -W 2 "$host" &>/dev/null; then
            echo "[OK] $host is reachable"
        else
            echo "[FAIL] $host is unreachable"
        fi
    done
}

check_connectivity
```

### Port Monitoring

```bash
#!/bin/bash

PORTS=(80 443 22 3306 6379)

check_ports() {
    echo "=== Open Ports ==="
    
    for port in "${PORTS[@]}"; do
        if netstat -tuln | grep -q ":$port "; then
            echo "[OK] Port $port is listening"
        else
            echo "[FAIL] Port $port is NOT listening"
        fi
    done
}

check_ports
```

---

## Automated Monitoring with Cron

### Setup Monitoring Cron Job

```bash
#!/bin/bash

# Install monitoring script
sudo cp system_health.sh /usr/local/bin/
sudo chmod +x /usr/local/bin/system_health.sh

# Add to crontab
(crontab -l 2>/dev/null; echo "*/5 * * * * /usr/local/bin/system_health.sh >> /var/log/health_check.log 2>&1") | crontab -

echo "Monitoring scheduled to run every 5 minutes"
```

---

## Best Practices

1. **Set appropriate thresholds** for your environment
2. **Monitor trends**, not just current values
3. **Implement alerting** for critical issues
4. **Log all monitoring data** for analysis
5. **Test monitoring scripts** regularly
6. **Document baseline metrics**
7. **Automate remediation** where possible

---

## Summary

- Monitor disk, memory, CPU, and services
- Set thresholds and alerts
- Automate health checks with cron
- Analyze logs for errors
- Check network connectivity
- Create comprehensive monitoring scripts

---

**Next Chapter:** [23 - Network Automation](23-network-automation.md)
