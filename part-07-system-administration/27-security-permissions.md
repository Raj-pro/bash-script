# Chapter 25: Security and Permissions

## Introduction

Security is paramount in system administration. This chapter covers file permissions, secure scripting practices, and automating security audits.

---

## Understanding Linux Permissions

### Permission Basics

```bash
# Permission format: rwxrwxrwx
# r = read (4), w = write (2), x = execute (1)
# First set: owner, second: group, third: others

# View permissions
ls -l file.txt
# -rw-r--r-- 1 user group 1234 Jan 01 12:00 file.txt

# View directory permissions
ls -ld /path/to/dir
```

### Numeric (Octal) Permissions

```bash
# Common permission modes
0644  # rw-r--r-- (files)
0755  # rwxr-xr-x (directories, executables)
0600  # rw------- (private files)
0700  # rwx------ (private directories)
0777  # rwxrwxrwx (world writable - dangerous!)

# Set permissions
chmod 644 file.txt
chmod 755 script.sh
```

### Symbolic Permissions

```bash
# Add execute for owner
chmod u+x script.sh

# Remove write from group
chmod g-w file.txt

# Add read for others
chmod o+r file.txt

# Set exact permissions
chmod u=rwx,g=rx,o=r file.txt

# Recursive
chmod -R 755 /var/www
```

---

## Ownership and Groups

### Changing Ownership

```bash
# Change owner
chown user file.txt

# Change owner and group
chown user:group file.txt

# Recursive
chown -R user:group /path/to/dir

# Change only group
chgrp group file.txt
```

### Ownership Management Script

```bash
#!/bin/bash

fix_web_permissions() {
    local web_root=$1
    local web_user="www-data"
    local web_group="www-data"
    
    echo "Fixing permissions for $web_root"
    
    # Set ownership
    chown -R "$web_user:$web_group" "$web_root"
    
    # Set directory permissions
    find "$web_root" -type d -exec chmod 755 {} \;
    
    # Set file permissions
    find "$web_root" -type f -exec chmod 644 {} \;
    
    echo "Permissions fixed"
}

fix_web_permissions "/var/www/html"
```

---

## Special Permissions

### Setuid, Setgid, and Sticky Bit

```bash
# Setuid (4000): Run as file owner
chmod 4755 /usr/bin/passwd

# Setgid (2000): Inherit group ownership
chmod 2755 /shared/directory

# Sticky bit (1000): Only owner can delete
chmod 1777 /tmp

# Combined
chmod 4755 file  # setuid + 755
chmod 2775 dir   # setgid + 775
chmod 1777 /tmp  # sticky + 777

# Symbolic
chmod u+s file   # setuid
chmod g+s dir    # setgid
chmod +t dir     # sticky bit
```

---

## umask - Default Permissions

### Understanding umask

```bash
# View current umask
umask

# Common umask values:
# 0022 -> files: 644, dirs: 755
# 0027 -> files: 640, dirs: 750
# 0077 -> files: 600, dirs: 700

# Set umask
umask 0022

# Set in script
umask 0077  # Secure: owner-only access by default
```

### umask in Scripts

```bash
#!/bin/bash

# Save original umask
old_umask=$(umask)

# Set restrictive umask
umask 0077

# Create files (will have 600 permissions)
touch sensitive_file.txt

# Create directories (will have 700 permissions)
mkdir private_dir

# Restore original umask
umask "$old_umask"
```

---

## Access Control Lists (ACL)

### Using ACLs

```bash
# Install ACL tools
sudo apt install acl  # Debian/Ubuntu
sudo yum install acl  # RHEL/CentOS

# Set ACL for specific user
setfacl -m u:username:rwx file.txt

# Set ACL for specific group
setfacl -m g:groupname:rx file.txt

# View ACLs
getfacl file.txt

# Remove specific ACL
setfacl -x u:username file.txt

# Remove all ACLs
setfacl -b file.txt

# Recursive
setfacl -R -m u:username:rwx /path/to/dir
```

---

## Secure Script Practices

### Secure Temp Files

```bash
#!/bin/bash

# INSECURE - predictable filename
temp_file="/tmp/myapp_$$.tmp"

# SECURE - use mktemp
temp_file=$(mktemp)
trap "rm -f '$temp_file'" EXIT

# Use the temp file
echo "data" > "$temp_file"

# Cleanup happens automatically on exit
```

### Input Validation

```bash
#!/bin/bash

# Validate user input
validate_input() {
    local input=$1
    
    # Check for null/empty
    if [ -z "$input" ]; then
        echo "Error: Input cannot be empty"
        return 1
    fi
    
    # Check for dangerous characters
    if [[ "$input" =~ [^a-zA-Z0-9._-] ]]; then
        echo "Error: Invalid characters in input"
        return 1
    fi
    
    # Check length
    if [ ${#input} -gt 255 ]; then
        echo "Error: Input too long"
        return 1
    fi
    
    return 0
}

# Usage
read -p "Enter filename: " filename
if validate_input "$filename"; then
    echo "Valid input: $filename"
else
    exit 1
fi
```

### Preventing Command Injection

```bash
#!/bin/bash

# INSECURE - command injection vulnerable
username=$1
grep "$username" /etc/passwd

# SECURE - quote variables
grep "$username" /etc/passwd

# BETTER - validate input first
if [[ "$username" =~ ^[a-z_][a-z0-9_-]*$ ]]; then
    grep "$username" /etc/passwd
else
    echo "Invalid username format"
    exit 1
fi
```

### Secure Password Handling

```bash
#!/bin/bash

# INSECURE - password in command line
mysql -u root -pPASSWORD

# SECURE - read from stdin
read -s -p "Enter password: " password
echo
mysql -u root -p"$password"

# BETTER - use password file
mysql --defaults-file=/etc/mysql/credentials.cnf

# BEST - use environment variable
export MYSQL_PWD="password"
mysql -u root
unset MYSQL_PWD
```

---

## File Permission Auditing

### Find Permission Issues

```bash
#!/bin/bash

audit_permissions() {
    echo "=== Permission Audit ==="
    
    # World-writable files
    echo "World-writable files:"
    find / -type f -perm -002 ! -path "/proc/*" 2>/dev/null
    
    # World-writable directories without sticky bit
    echo -e "\nWorld-writable directories without sticky bit:"
    find / -type d -perm -002 ! -perm -1000 ! -path "/proc/*" 2>/dev/null
    
    # Files with setuid/setgid
    echo -e "\nSetuid/Setgid files:"
    find / -type f \( -perm -4000 -o -perm -2000 \) ! -path "/proc/*" 2>/dev/null
    
    # Files owned by nobody
    echo -e "\nFiles owned by nobody:"
    find / -nouser -o -nogroup 2>/dev/null | head -20
}

audit_permissions > permission_audit.txt
```

### Check Script Security

```bash
#!/bin/bash

check_script_security() {
    local script=$1
    
    echo "Checking security of: $script"
    
    # Check ownership
    owner=$(stat -c '%U' "$script")
    if [ "$owner" != "root" ]; then
        echo "[WARN] Script not owned by root: $owner"
    fi
    
    # Check permissions
    perms=$(stat -c '%a' "$script")
    if [ "$perms" != "700" ] && [ "$perms" != "750" ] && [ "$perms" != "755" ]; then
        echo "[WARN] Insecure permissions: $perms"
    fi
    
    # Check for hardcoded passwords
    if grep -iE 'password|passwd|pwd' "$script" | grep -v '^#' > /dev/null; then
        echo "[WARN] Possible hardcoded password found"
    fi
    
    # Check for world-writable paths
    if grep -oE '(/tmp|/var/tmp)/[^/]+' "$script" | grep -v '\$(mktemp)' > /dev/null; then
        echo "[WARN] Potentially insecure temp file usage"
    fi
}

check_script_security "/path/to/script.sh"
```

---

## Detecting Unauthorized Access

### Monitor Failed Login Attempts

```bash
#!/bin/bash

THRESHOLD=5
LOG_FILE="/var/log/auth.log"

check_failed_logins() {
    echo "=== Failed Login Attempts ==="
    
    # Last hour failed logins
    grep "Failed password" "$LOG_FILE" | \
    grep "$(date '+%b %e %H')" | \
    awk '{print $(NF-5)}' | \
    sort | uniq -c | sort -rn | \
    while read count user; do
        if [ "$count" -gt "$THRESHOLD" ]; then
            echo "[ALERT] User $user: $count failed attempts"
            # Block IP or send alert
        fi
    done
}

check_failed_logins
```

### Monitor File Changes

```bash
#!/bin/bash

WATCH_DIRS=("/etc" "/usr/bin" "/usr/sbin")
BASELINE="/var/lib/file_integrity/baseline.txt"

create_baseline() {
    mkdir -p "$(dirname "$BASELINE")"
    
    for dir in "${WATCH_DIRS[@]}"; do
        find "$dir" -type f -exec sha256sum {} \;
    done > "$BASELINE"
    
    echo "Baseline created"
}

check_integrity() {
    local current="/tmp/current_$$.txt"
    
    for dir in "${WATCH_DIRS[@]}"; do
        find "$dir" -type f -exec sha256sum {} \;
    done > "$current"
    
    # Compare with baseline
    if diff "$BASELINE" "$current" > /dev/null; then
        echo "No changes detected"
    else
        echo "ALERT: File changes detected!"
        diff "$BASELINE" "$current"
    fi
    
    rm -f "$current"
}

# Create baseline if doesn't exist
[ ! -f "$BASELINE" ] && create_baseline

# Check integrity
check_integrity
```

---

## SSH Security

### SSH Hardening Script

```bash
#!/bin/bash

SSHD_CONFIG="/etc/ssh/sshd_config"
BACKUP_CONFIG="${SSHD_CONFIG}.backup.$(date +%Y%m%d)"

harden_ssh() {
    echo "Hardening SSH configuration..."
    
    # Backup current config
    cp "$SSHD_CONFIG" "$BACKUP_CONFIG"
    
    # Disable root login
    sed -i 's/^PermitRootLogin.*/PermitRootLogin no/' "$SSHD_CONFIG"
    
    # Disable password authentication (use keys only)
    sed -i 's/^PasswordAuthentication.*/PasswordAuthentication no/' "$SSHD_CONFIG"
    
    # Disable empty passwords
    sed -i 's/^PermitEmptyPasswords.*/PermitEmptyPasswords no/' "$SSHD_CONFIG"
    
    # Change default port (optional)
    # sed -i 's/^#Port 22/Port 2222/' "$SSHD_CONFIG"
    
    # Limit authentication attempts
    sed -i 's/^#MaxAuthTries.*/MaxAuthTries 3/' "$SSHD_CONFIG"
    
    # Test configuration
    if sshd -t; then
        echo "Configuration valid"
        systemctl restart sshd
        echo "SSH hardened successfully"
    else
        echo "Configuration invalid! Restoring backup"
        cp "$BACKUP_CONFIG" "$SSHD_CONFIG"
        exit 1
    fi
}

harden_ssh
```

---

## Firewall Automation

### Basic iptables Rules

```bash
#!/bin/bash

# Flush existing rules
iptables -F

# Default policies
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Allow loopback
iptables -A INPUT -i lo -j ACCEPT

# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP/HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Save rules
iptables-save > /etc/iptables/rules.v4

echo "Firewall configured"
```

---

## Best Practices

1. **Use least privilege** principle
2. **Regularly audit permissions**
3. **Validate all user input**
4. **Use secure temp files** (mktemp)
5. **Never hardcode passwords**
6. **Monitor for unauthorized access**
7. **Keep systems updated**
8. **Use SSH keys** instead of passwords

---

## Summary

- Understand and manage file permissions (chmod, chown)
- Use umask for default permissions
- Implement secure coding practices
- Audit permissions regularly
- Monitor for security violations
- Harden SSH and firewall configurations

---

**Next Chapter:** Part 8 - Bash in DevOps and Automation
