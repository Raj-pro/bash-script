# Chapter 25: Network Automation

## Introduction

Network automation with Bash enables you to manage connectivity, perform scans, and automate downloads/uploads. This chapter covers practical network scripting techniques.

---

## Network Connectivity Testing

### Basic Ping Tests

```bash
#!/bin/bash

# Single host ping
ping -c 4 google.com

# Ping with timeout
ping -c 1 -W 2 192.168.1.1

# Ping and check status
if ping -c 1 -W 2 google.com &>/dev/null; then
    echo "Host is reachable"
else
    echo "Host is unreachable"
fi
```

### Ping Sweep Script

```bash
#!/bin/bash

network="192.168.1"

echo "Scanning network $network.0/24"

for i in {1..254}; do
    ip="$network.$i"
    
    ping -c 1 -W 1 "$ip" &>/dev/null
    
    if [ $? -eq 0 ]; then
        echo "$ip is UP"
    fi
done
```

### Parallel Ping Sweep

```bash
#!/bin/bash

network="192.168.1"
MAX_JOBS=50

ping_host() {
    local ip=$1
    if ping -c 1 -W 1 "$ip" &>/dev/null; then
        echo "$ip is UP"
    fi
}

# Wait for job slots
wait_for_jobs() {
    while [ $(jobs -r | wc -l) -ge $MAX_JOBS ]; do
        sleep 0.1
    done
}

echo "Scanning $network.0/24 (parallel)"

for i in {1..254}; do
    wait_for_jobs
    ping_host "$network.$i" &
done

wait
echo "Scan complete"
```

---

## Port Scanning

### Check Single Port

```bash
#!/bin/bash

check_port() {
    local host=$1
    local port=$2
    local timeout=2
    
    if timeout $timeout bash -c "echo >/dev/tcp/$host/$port" 2>/dev/null; then
        echo "Port $port is OPEN on $host"
        return 0
    else
        echo "Port $port is CLOSED on $host"
        return 1
    fi
}

# Usage
check_port "192.168.1.1" 80
```

### Port Range Scanner

```bash
#!/bin/bash

scan_ports() {
    local host=$1
    local start_port=$2
    local end_port=$3
    
    echo "Scanning $host from port $start_port to $end_port"
    
    for port in $(seq $start_port $end_port); do
        if timeout 1 bash -c "echo >/dev/tcp/$host/$port" 2>/dev/null; then
            echo "Port $port: OPEN"
        fi
    done
}

# Usage
scan_ports "192.168.1.1" 1 1000
```

### Common Ports Scanner

```bash
#!/bin/bash

host=$1

# Common ports
ports=(20 21 22 23 25 53 80 110 143 443 445 3306 3389 5432 6379 8080 27017)

echo "Scanning common ports on $host"

for port in "${ports[@]}"; do
    if timeout 1 bash -c "echo >/dev/tcp/$host/$port" 2>/dev/null; then
        # Identify service
        case $port in
            20|21) service="FTP" ;;
            22) service="SSH" ;;
            23) service="Telnet" ;;
            25) service="SMTP" ;;
            53) service="DNS" ;;
            80) service="HTTP" ;;
            110) service="POP3" ;;
            143) service="IMAP" ;;
            443) service="HTTPS" ;;
            445) service="SMB" ;;
            3306) service="MySQL" ;;
            3389) service="RDP" ;;
            5432) service="PostgreSQL" ;;
            6379) service="Redis" ;;
            8080) service="HTTP-Alt" ;;
            27017) service="MongoDB" ;;
            *) service="Unknown" ;;
        esac
        
        echo "Port $port: OPEN [$service]"
    fi
done
```

---

## Using curl for Web Automation

### Basic curl Usage

```bash
# GET request
curl https://api.github.com

# Save to file
curl -o output.html https://example.com

# Follow redirects
curl -L https://example.com

# Show headers
curl -I https://example.com

# Verbose output
curl -v https://example.com
```

### POST Requests

```bash
# POST with data
curl -X POST -d "name=John&age=30" https://api.example.com/users

# POST JSON data
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"name":"John","age":30}' \
  https://api.example.com/users

# POST from file
curl -X POST \
  -H "Content-Type: application/json" \
  -d @data.json \
  https://api.example.com/users
```

### API Automation

```bash
#!/bin/bash

API_URL="https://api.example.com"
API_KEY="your_api_key"

# GET request with authentication
get_users() {
    curl -s -H "Authorization: Bearer $API_KEY" \
        "$API_URL/users" | jq '.'
}

# Create user
create_user() {
    local name=$1
    local email=$2
    
    curl -s -X POST \
        -H "Authorization: Bearer $API_KEY" \
        -H "Content-Type: application/json" \
        -d "{\"name\":\"$name\",\"email\":\"$email\"}" \
        "$API_URL/users" | jq '.'
}

# Delete user
delete_user() {
    local user_id=$1
    
    curl -s -X DELETE \
        -H "Authorization: Bearer $API_KEY" \
        "$API_URL/users/$user_id"
}

# Usage
get_users
create_user "John Doe" "john@example.com"
```

### Health Check Script

```bash
#!/bin/bash

URLS=(
    "https://example.com"
    "https://api.example.com"
    "https://cdn.example.com"
)

check_url() {
    local url=$1
    local status=$(curl -s -o /dev/null -w "%{http_code}" "$url")
    local response_time=$(curl -s -o /dev/null -w "%{time_total}" "$url")
    
    if [ "$status" -eq 200 ]; then
        echo "[OK] $url - Status: $status, Time: ${response_time}s"
    else
        echo "[FAIL] $url - Status: $status"
        # Send alert
    fi
}

for url in "${URLS[@]}"; do
    check_url "$url"
done
```

---

## File Downloads with wget/curl

### wget Basics

```bash
# Download file
wget https://example.com/file.zip

# Download to specific name
wget -O output.zip https://example.com/file.zip

# Download in background
wget -b https://example.com/largefile.iso

# Resume download
wget -c https://example.com/file.zip

# Download multiple files
wget -i urls.txt

# Mirror website
wget --mirror --convert-links --adjust-extension --page-requisites --no-parent https://example.com
```

### Batch Download Script

```bash
#!/bin/bash

URLS_FILE="downloads.txt"
DEST_DIR="/downloads"

mkdir -p "$DEST_DIR"

while IFS= read -r url; do
    echo "Downloading: $url"
    
    filename=$(basename "$url")
    
    if wget -q -O "$DEST_DIR/$filename" "$url"; then
        echo "[OK] Downloaded: $filename"
    else
        echo "[FAIL] Failed to download: $url"
    fi
done < "$URLS_FILE"

echo "All downloads complete"
```

---

## Remote File Transfer

### SCP (Secure Copy)

```bash
# Copy to remote
scp file.txt user@server:/remote/path/

# Copy from remote
scp user@server:/remote/file.txt /local/path/

# Copy directory recursively
scp -r /local/dir user@server:/remote/path/

# Copy with specific port
scp -P 2222 file.txt user@server:/path/

# Copy multiple files
scp file1.txt file2.txt user@server:/path/
```

### Automated SCP Script

```bash
#!/bin/bash

REMOTE_HOST="user@server.com"
REMOTE_PATH="/backup/"
LOCAL_PATH="/data/"

backup_to_remote() {
    local files=$1
    
    echo "Backing up to $REMOTE_HOST:$REMOTE_PATH"
    
    if scp -r "$files" "$REMOTE_HOST:$REMOTE_PATH"; then
        echo "[OK] Backup successful"
    else
        echo "[FAIL] Backup failed"
        return 1
    fi
}

# Backup specific directory
backup_to_remote "$LOCAL_PATH/*"
```

### rsync for Efficient Transfers

```bash
# Basic sync
rsync -avz /source/ user@server:/dest/

# Sync with progress
rsync -avz --progress /source/ user@server:/dest/

# Delete files not in source
rsync -avz --delete /source/ user@server:/dest/

# Dry run (test without changes)
rsync -avz --dry-run /source/ user@server:/dest/

# Exclude patterns
rsync -avz --exclude='*.log' --exclude='tmp/' /source/ user@server:/dest/
```

### Incremental Backup with rsync

```bash
#!/bin/bash

SOURCE="/data/"
DEST="user@backup-server:/backups/"
LOG_FILE="/var/log/backup.log"
DATE=$(date +%Y%m%d_%H%M%S)

backup() {
    echo "[$DATE] Starting backup..." | tee -a "$LOG_FILE"
    
    rsync -avz \
        --delete \
        --exclude='*.tmp' \
        --exclude='.cache' \
        --log-file="$LOG_FILE" \
        "$SOURCE" \
        "$DEST"
    
    if [ $? -eq 0 ]; then
        echo "[$DATE] Backup completed successfully" | tee -a "$LOG_FILE"
    else
        echo "[$DATE] Backup failed!" | tee -a "$LOG_FILE"
        # Send alert
        return 1
    fi
}

backup
```

---

## DNS Operations

### DNS Lookup

```bash
# Basic lookup
nslookup example.com

# Dig command
dig example.com

# Get A records
dig example.com A

# Get MX records
dig example.com MX

# Short answer
dig +short example.com

# Reverse DNS
dig -x 8.8.8.8
```

### DNS Check Script

```bash
#!/bin/bash

check_dns() {
    local domain=$1
    
    echo "=== DNS Information for $domain ==="
    
    # A records
    echo "A Records:"
    dig +short "$domain" A
    
    # MX records
    echo -e "\nMX Records:"
    dig +short "$domain" MX
    
    # NS records
    echo -e "\nName Servers:"
    dig +short "$domain" NS
    
    # TXT records
    echo -e "\nTXT Records:"
    dig +short "$domain" TXT
}

check_dns "example.com"
```

---

## SSH Automation

### SSH Key-Based Authentication Setup

```bash
#!/bin/bash

setup_ssh_key() {
    local user=$1
    local host=$2
    
    # Generate key if doesn't exist
    if [ ! -f ~/.ssh/id_rsa ]; then
        ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""
    fi
    
    # Copy to remote
    ssh-copy-id "$user@$host"
    
    echo "SSH key authentication set up for $user@$host"
}

setup_ssh_key "admin" "server.com"
```

### Execute Remote Commands

```bash
#!/bin/bash

SERVERS=("server1.com" "server2.com" "server3.com")
USER="admin"

execute_remote() {
    local command=$1
    
    for server in "${SERVERS[@]}"; do
        echo "Executing on $server:"
        ssh "$USER@$server" "$command"
        echo "---"
    done
}

# Usage
execute_remote "uptime"
execute_remote "df -h"
```

---

## Network Information

### Get Network Interfaces

```bash
#!/bin/bash

# List interfaces
ip addr show

# Get IP address
hostname -I

# Interface details
for iface in $(ip -o link show | awk -F': ' '{print $2}'); do
    echo "Interface: $iface"
    ip addr show "$iface" | grep inet
    echo
done
```

### Network Statistics

```bash
#!/bin/bash

echo "=== Network Statistics ==="

# Active connections
echo "Active Connections:"
netstat -an | grep ESTABLISHED | wc -l

# Listening ports
echo -e "\nListening Ports:"
netstat -tuln

# Network traffic
echo -e "\nNetwork Traffic:"
cat /proc/net/dev | column -t
```

---

## Best Practices

1. **Use timeouts** for network operations
2. **Implement retry logic** for unreliable connections
3. **Validate responses** from network requests
4. **Use SSH keys** instead of passwords
5. **Log all network operations**
6. **Handle errors gracefully**
7. **Set appropriate connection limits**

---

## Summary

- Automate connectivity tests with ping
- Scan ports with bash TCP connections
- Use curl/wget for HTTP automation
- Transfer files with scp/rsync
- Execute remote commands via SSH
- Monitor DNS and network status

---

**Next Chapter:** [26 - Backup and Recovery Automation](26-backup-recovery-automation.md)
