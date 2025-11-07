# Chapter 21: User and Group Management

## Introduction

Automating user and group management is crucial for system administration. This chapter covers creating, modifying, and managing users and groups through Bash scripts.

---

## Understanding Users and Groups

### User Account Basics

```bash
# User information files
/etc/passwd    # User account information
/etc/shadow    # Encrypted passwords
/etc/group     # Group information
/etc/gshadow   # Encrypted group passwords

# /etc/passwd format:
# username:x:UID:GID:comment:home:shell
# Example: john:x:1001:1001:John Doe:/home/john:/bin/bash
```

### Viewing User Information

```bash
# Current user
whoami
id

# User details
id username
finger username  # May need installation

# List all users
cat /etc/passwd | cut -d: -f1

# List logged-in users
who
w
users

# Last login information
last
lastlog
```

---

## Creating Users

### Using useradd

```bash
# Basic user creation
sudo useradd john

# Create with home directory
sudo useradd -m john

# Specify shell
sudo useradd -m -s /bin/bash john

# Set home directory
sudo useradd -m -d /home/custom john

# Add to groups
sudo useradd -m -G sudo,docker john

# Set expiry date
sudo useradd -m -e 2025-12-31 john

# Complete example
sudo useradd -m -s /bin/bash -c "John Doe" -G sudo,users john
```

### Using adduser (Interactive)

```bash
# More user-friendly (Debian/Ubuntu)
sudo adduser john

# Non-interactive mode
sudo adduser --disabled-password --gecos "John Doe" john
```

### Setting Passwords

```bash
# Interactive password set
sudo passwd john

# Set password non-interactively
echo "john:password123" | sudo chpasswd

# Using stdin
echo "password123" | sudo passwd --stdin john

# Force password change on next login
sudo passwd -e john

# Generate random password
PASS=$(openssl rand -base64 12)
echo "john:$PASS" | sudo chpasswd
echo "Password for john: $PASS"
```

---

## Modifying Users

### Using usermod

```bash
# Change username
sudo usermod -l newname oldname

# Change home directory
sudo usermod -d /new/home -m username

# Change shell
sudo usermod -s /bin/zsh username

# Lock account
sudo usermod -L username

# Unlock account
sudo usermod -U username

# Add to groups
sudo usermod -aG sudo,docker username

# Set expiry date
sudo usermod -e 2025-12-31 username

# Change UID
sudo usermod -u 2000 username
```

### Modifying User Properties

```bash
# Change comment/GECOS
sudo usermod -c "New Description" username

# Change primary group
sudo usermod -g groupname username

# Move home directory
sudo usermod -d /new/path -m username
```

---

## Deleting Users

### Using userdel

```bash
# Delete user (keep home directory)
sudo userdel username

# Delete user and home directory
sudo userdel -r username

# Force deletion (even if logged in)
sudo userdel -f username
```

### Cleanup After Deletion

```bash
#!/bin/bash

delete_user_complete() {
    local username=$1
    
    # Kill user processes
    sudo pkill -u "$username"
    
    # Delete user and home
    sudo userdel -r "$username"
    
    # Remove from additional groups
    sudo gpasswd -M "" "$username" 2>/dev/null
    
    # Clean up cron jobs
    sudo crontab -r -u "$username" 2>/dev/null
    
    # Remove mail spool
    sudo rm -f "/var/mail/$username"
    
    echo "User $username completely removed"
}
```

---

## Group Management

### Creating Groups

```bash
# Create group
sudo groupadd developers

# Create with specific GID
sudo groupadd -g 5000 developers

# Create system group
sudo groupadd -r systemgroup
```

### Managing Group Membership

```bash
# Add user to group
sudo usermod -aG groupname username
sudo gpasswd -a username groupname

# Remove user from group
sudo gpasswd -d username groupname

# Set group members (replaces existing)
sudo gpasswd -M user1,user2,user3 groupname

# List group members
getent group groupname
grep groupname /etc/group
```

### Deleting Groups

```bash
# Delete group
sudo groupdel groupname

# Note: Can't delete if it's a primary group for any user
```

---

## Automated User Creation

### Single User Creation Script

```bash
#!/bin/bash

create_user() {
    local username=$1
    local fullname=$2
    local groups=$3
    
    # Check if user exists
    if id "$username" &>/dev/null; then
        echo "User $username already exists"
        return 1
    fi
    
    # Create user
    sudo useradd -m -s /bin/bash -c "$fullname" "$username"
    
    # Add to groups if specified
    if [ -n "$groups" ]; then
        sudo usermod -aG "$groups" "$username"
    fi
    
    # Generate random password
    local password=$(openssl rand -base64 12)
    echo "$username:$password" | sudo chpasswd
    
    # Force password change
    sudo passwd -e "$username"
    
    # Create user directories
    sudo mkdir -p "/home/$username"/{Documents,Downloads,Projects}
    sudo chown -R "$username:$username" "/home/$username"
    
    echo "User created: $username"
    echo "Password: $password"
    echo "Groups: $(groups $username)"
}

# Usage
create_user "jdoe" "John Doe" "sudo,developers"
```

### Bulk User Creation

```bash
#!/bin/bash

# users.txt format:
# username,fullname,groups
# jdoe,John Doe,developers
# jsmith,Jane Smith,developers,sudo

USERS_FILE="users.txt"
LOG_FILE="user_creation.log"

bulk_create_users() {
    while IFS=, read -r username fullname groups; do
        # Skip empty lines and comments
        [[ -z "$username" || "$username" =~ ^# ]] && continue
        
        echo "Creating user: $username" | tee -a "$LOG_FILE"
        
        # Check if exists
        if id "$username" &>/dev/null; then
            echo "  [SKIP] User exists" | tee -a "$LOG_FILE"
            continue
        fi
        
        # Create user
        if sudo useradd -m -s /bin/bash -c "$fullname" "$username"; then
            echo "  [OK] User created" | tee -a "$LOG_FILE"
            
            # Add to groups
            if [ -n "$groups" ]; then
                sudo usermod -aG "$groups" "$username"
                echo "  [OK] Added to groups: $groups" | tee -a "$LOG_FILE"
            fi
            
            # Generate password
            password=$(openssl rand -base64 12)
            echo "$username:$password" | sudo chpasswd
            sudo passwd -e "$username"
            
            echo "  [OK] Password set (temporary): $password" | tee -a "$LOG_FILE"
        else
            echo "  [ERROR] Failed to create user" | tee -a "$LOG_FILE"
        fi
        
    done < "$USERS_FILE"
}

bulk_create_users
```

---

## User Auditing and Reporting

### List All Users

```bash
#!/bin/bash

list_users() {
    local min_uid=1000
    
    echo "Regular Users:"
    echo "Username:UID:GID:Home:Shell"
    echo "================================"
    
    while IFS=: read -r username x uid gid comment home shell; do
        if [ "$uid" -ge $min_uid ] && [ "$uid" -ne 65534 ]; then
            echo "$username:$uid:$gid:$home:$shell"
        fi
    done < /etc/passwd
}

list_users
```

### User Activity Report

```bash
#!/bin/bash

user_activity_report() {
    echo "=== User Activity Report ==="
    echo "Generated: $(date)"
    echo
    
    # Logged in users
    echo "Currently Logged In:"
    w | tail -n +3
    echo
    
    # Recent logins
    echo "Recent Logins (last 10):"
    last -n 10
    echo
    
    # Failed logins
    echo "Failed Login Attempts:"
    sudo lastb -n 10
    echo
    
    # Password expiry
    echo "Password Expiry Status:"
    while IFS=: read -r username x uid x; do
        if [ "$uid" -ge 1000 ] && [ "$uid" -ne 65534 ]; then
            expiry=$(sudo chage -l "$username" 2>/dev/null | grep "Password expires" | cut -d: -f2)
            echo "$username: $expiry"
        fi
    done < /etc/passwd
}

user_activity_report > user_activity_$(date +%Y%m%d).txt
```

### Inactive User Detection

```bash
#!/bin/bash

find_inactive_users() {
    local days=${1:-90}
    
    echo "Users inactive for $days+ days:"
    
    while IFS=: read -r username x uid x; do
        if [ "$uid" -ge 1000 ] && [ "$uid" -ne 65534 ]; then
            last_login=$(last -n 1 "$username" | head -1 | awk '{print $4,$5,$6}')
            
            if [ "$last_login" = "Never Logged In" ] || [ -z "$last_login" ]; then
                echo "  $username: Never logged in"
            else
                # Calculate days since last login
                last_timestamp=$(date -d "$last_login" +%s 2>/dev/null)
                if [ -n "$last_timestamp" ]; then
                    days_since=$((( $(date +%s) - last_timestamp ) / 86400))
                    if [ "$days_since" -gt "$days" ]; then
                        echo "  $username: $days_since days ago"
                    fi
                fi
            fi
        fi
    done < /etc/passwd
}

find_inactive_users 90
```

---

## Password Management

### Password Policy Script

```bash
#!/bin/bash

set_password_policy() {
    local username=$1
    
    # Password expires in 90 days
    sudo chage -M 90 "$username"
    
    # Warning 7 days before expiry
    sudo chage -W 7 "$username"
    
    # Minimum 1 day between password changes
    sudo chage -m 1 "$username"
    
    # Account expires after 180 days of inactivity
    sudo chage -I 180 "$username"
    
    echo "Password policy set for $username"
    sudo chage -l "$username"
}
```

### Force Password Reset

```bash
#!/bin/bash

force_password_reset() {
    local username=$1
    
    # Expire password
    sudo passwd -e "$username"
    
    # Set expiry to today
    sudo chage -d 0 "$username"
    
    echo "Password reset required for $username on next login"
}
```

---

## Sudo Access Management

### Adding User to Sudoers

```bash
#!/bin/bash

grant_sudo() {
    local username=$1
    
    # Add to sudo group (Debian/Ubuntu)
    sudo usermod -aG sudo "$username"
    
    # Or add to wheel group (RHEL/CentOS)
    # sudo usermod -aG wheel "$username"
    
    echo "$username granted sudo access"
}
```

### Custom Sudoers Entry

```bash
#!/bin/bash

create_limited_sudo() {
    local username=$1
    local commands=$2
    local sudoers_file="/etc/sudoers.d/$username"
    
    # Create sudoers file
    echo "$username ALL=(ALL) NOPASSWD: $commands" | sudo tee "$sudoers_file"
    
    # Set proper permissions
    sudo chmod 0440 "$sudoers_file"
    
    # Validate
    sudo visudo -c -f "$sudoers_file"
    
    echo "Limited sudo access created for $username"
}

# Usage: Allow user to restart nginx
create_limited_sudo "webadmin" "/usr/sbin/service nginx *"
```

---

## Practical Examples

### Complete User Onboarding Script

```bash
#!/bin/bash

onboard_user() {
    local username=$1
    local fullname=$2
    local department=$3
    local manager_email=$4
    
    echo "=== Onboarding User: $username ==="
    
    # Create user
    sudo useradd -m -s /bin/bash -c "$fullname" "$username"
    
    # Generate password
    password=$(openssl rand -base64 16)
    echo "$username:$password" | sudo chpasswd
    sudo passwd -e "$username"
    
    # Add to department group
    if ! getent group "$department" &>/dev/null; then
        sudo groupadd "$department"
    fi
    sudo usermod -aG "$department" "$username"
    
    # Set up home directory
    sudo mkdir -p "/home/$username"/{Documents,Downloads,Projects}
    sudo cp /etc/skel/.bashrc "/home/$username/"
    sudo chown -R "$username:$username" "/home/$username"
    
    # Set password policy
    sudo chage -M 90 -m 1 -W 7 "$username"
    
    # Create welcome message
    cat << EOF | sudo tee "/home/$username/WELCOME.txt"
Welcome $fullname!

Your account has been created.
Username: $username
Department: $department

Please change your password on first login.

For support, contact: $manager_email
EOF
    
    sudo chown "$username:$username" "/home/$username/WELCOME.txt"
    
    # Log the creation
    echo "$(date): Created user $username ($fullname) in $department" >> /var/log/user_management.log
    
    # Send notification (pseudo-code)
    # send_email "$manager_email" "New user created: $username" "Password: $password"
    
    echo "User $username onboarded successfully"
    echo "Temporary password: $password"
}

# Usage
onboard_user "jdoe" "John Doe" "engineering" "manager@company.com"
```

### User Offboarding Script

```bash
#!/bin/bash

offboard_user() {
    local username=$1
    local backup_dir="/home/archived_users"
    
    echo "=== Offboarding User: $username ==="
    
    # Verify user exists
    if ! id "$username" &>/dev/null; then
        echo "User $username does not exist"
        return 1
    fi
    
    # Lock account
    sudo usermod -L "$username"
    sudo usermod -s /bin/false "$username"
    echo "Account locked"
    
    # Kill all user processes
    sudo pkill -u "$username"
    echo "Terminated all processes"
    
    # Backup home directory
    sudo mkdir -p "$backup_dir"
    sudo tar -czf "$backup_dir/${username}_$(date +%Y%m%d).tar.gz" "/home/$username"
    echo "Home directory backed up"
    
    # Remove cron jobs
    sudo crontab -r -u "$username" 2>/dev/null
    echo "Cron jobs removed"
    
    # Generate exit report
    cat << EOF | sudo tee "$backup_dir/${username}_report.txt"
User Offboarding Report
=======================
Username: $username
Date: $(date)
Last Login: $(last -n 1 "$username" | head -1)
Groups: $(groups "$username")
Files backed up to: $backup_dir/${username}_$(date +%Y%m%d).tar.gz
EOF
    
    # Optional: Delete user (uncomment if needed)
    # sudo userdel "$username"
    
    echo "User $username offboarded successfully"
}

# Usage
offboard_user "jdoe"
```

---

## Best Practices

1. **Always use strong passwords** or key-based authentication
2. **Implement password policies** with `chage`
3. **Use groups** for permission management
4. **Audit user activity** regularly
5. **Back up home directories** before deletion
6. **Document user changes** in logs
7. **Automate repetitive tasks** with scripts
8. **Follow principle of least privilege**

---

## Summary

- Use `useradd`, `usermod`, `userdel` for user management
- Use `groupadd`, `gpasswd`, `groupdel` for groups
- Automate bulk operations with scripts
- Implement password policies with `chage`
- Audit users regularly for security
- Create comprehensive onboarding/offboarding procedures

---

## Practice Exercises

1. Create a script to provision 50 users from a CSV file
2. Build a user audit tool that checks for security violations
3. Implement an automated password expiry notification system
4. Create a sudo access management interface

---

**Next Chapter:** [22 - System Monitoring and Health Checks](22-system-monitoring-health-checks.md)
