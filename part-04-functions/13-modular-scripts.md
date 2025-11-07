# Chapter 13: Modular Scripts

## Introduction

As your Bash scripts grow in complexity, organizing code into modular, reusable components becomes essential. This chapter teaches you how to structure professional Bash projects using source files, libraries, and maintainable architectures that scale from simple utilities to enterprise automation systems.

## Table of Contents
- [Why Modular Scripts?](#why-modular-scripts)
- [Using Source and Dot Operator](#using-source-and-dot-operator)
- [Creating Function Libraries](#creating-function-libraries)
- [Project Structure](#project-structure)
- [Import Patterns](#import-patterns)
- [Configuration Management](#configuration-management)
- [Error Handling Across Modules](#error-handling-across-modules)
- [Best Practices](#best-practices)

---

## Why Modular Scripts?

### Benefits of Modular Design

| Benefit | Description |
|---------|-------------|
| **Reusability** | Write once, use in multiple scripts |
| **Maintainability** | Update logic in one place |
| **Testing** | Test modules independently |
| **Collaboration** | Multiple developers can work on different modules |
| **Organization** | Logical separation of concerns |
| **Debugging** | Easier to isolate and fix issues |

### Monolithic vs Modular

```bash
# ❌ MONOLITHIC: Everything in one file (hard to maintain)
#!/bin/bash
# 500 lines of functions and logic...
validate_email() { ... }
send_notification() { ... }
backup_database() { ... }
check_disk_space() { ... }
# ... hundreds more lines ...
main() { ... }

# ✅ MODULAR: Organized into logical modules
#!/bin/bash
source lib/validators.sh    # Email, IP, URL validation
source lib/notifications.sh  # Email, Slack, SMS
source lib/backup.sh         # Database, file backups
source lib/monitoring.sh     # Disk, CPU, memory checks

main() {
    validate_email "$email" || exit 1
    backup_database "production"
    send_notification "Backup complete"
}
```

---

## Using Source and Dot Operator

The `source` command (or `.` operator) executes another script in the current shell environment:

### Basic Syntax

```bash
# Both are equivalent
source filename.sh
. filename.sh
```

### Simple Example

```bash
# lib/greetings.sh
hello() {
    echo "Hello, $1!"
}

goodbye() {
    echo "Goodbye, $1!"
}

# main.sh
#!/bin/bash
source lib/greetings.sh

hello "Alice"
goodbye "Bob"
```

### Source vs Execute

```bash
# lib/test.sh
#!/bin/bash
export MY_VAR="Hello"
echo "Script executed"

# Comparison
./lib/test.sh          # Runs in subshell, MY_VAR not available here
source lib/test.sh     # Runs in current shell, MY_VAR is available

echo $MY_VAR           # Only works after 'source'
```

### Conditional Sourcing

```bash
# Source only if file exists
if [ -f "lib/utils.sh" ]; then
    source lib/utils.sh
else
    echo "Error: utils.sh not found" >&2
    exit 1
fi

# Or use short-circuit
[ -f "lib/utils.sh" ] && source lib/utils.sh || {
    echo "Error: utils.sh not found" >&2
    exit 1
}
```

---

## Creating Function Libraries

### Basic Library Structure

```bash
# lib/string_utils.sh
#!/bin/bash

##
# String utility functions library
##

# Convert string to uppercase
to_upper() {
    echo "$1" | tr '[:lower:]' '[:upper:]'
}

# Convert string to lowercase
to_lower() {
    echo "$1" | tr '[:upper:]' '[:lower:]'
}

# Reverse a string
reverse_string() {
    echo "$1" | rev
}

# Count characters
count_chars() {
    echo -n "$1" | wc -c
}

# Trim whitespace
trim() {
    echo "$1" | sed 's/^[[:space:]]*//;s/[[:space:]]*$//'
}
```

### Using the Library

```bash
#!/bin/bash
# main.sh

source lib/string_utils.sh

# Use library functions
name="alice"
echo "Original: $name"
echo "Uppercase: $(to_upper "$name")"
echo "Reversed: $(reverse_string "$name")"
echo "Length: $(count_chars "$name")"
```

### Math Library Example

```bash
# lib/math.sh
#!/bin/bash

##
# Mathematical operations library
##

# Addition
add() {
    echo $(( $1 + $2 ))
}

# Subtraction
subtract() {
    echo $(( $1 - $2 ))
}

# Multiplication
multiply() {
    echo $(( $1 * $2 ))
}

# Division with error handling
divide() {
    if [ "$2" -eq 0 ]; then
        echo "Error: Division by zero" >&2
        return 1
    fi
    echo $(( $1 / $2 ))
}

# Calculate percentage
percentage() {
    local value=$1
    local total=$2
    echo "scale=2; ($value / $total) * 100" | bc
}

# Is number prime?
is_prime() {
    local num=$1
    if [ $num -lt 2 ]; then
        return 1
    fi
    
    for ((i=2; i*i<=num; i++)); do
        if [ $((num % i)) -eq 0 ]; then
            return 1
        fi
    done
    return 0
}
```

### Validation Library

```bash
# lib/validators.sh
#!/bin/bash

##
# Input validation functions
##

# Validate email address
is_valid_email() {
    local email="$1"
    [[ "$email" =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]
}

# Validate IP address
is_valid_ip() {
    local ip="$1"
    [[ "$ip" =~ ^([0-9]{1,3}\.){3}[0-9]{1,3}$ ]] || return 1
    
    IFS='.' read -ra octets <<< "$ip"
    for octet in "${octets[@]}"; do
        [ "$octet" -gt 255 ] && return 1
    done
    return 0
}

# Validate URL
is_valid_url() {
    local url="$1"
    [[ "$url" =~ ^https?://[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}(/.*)?$ ]]
}

# Validate port number
is_valid_port() {
    local port="$1"
    [[ "$port" =~ ^[0-9]+$ ]] && [ "$port" -ge 1 ] && [ "$port" -le 65535 ]
}

# Validate phone number (simple US format)
is_valid_phone() {
    local phone="$1"
    [[ "$phone" =~ ^[0-9]{3}-[0-9]{3}-[0-9]{4}$ ]]
}

# Check if string is empty
is_empty() {
    [ -z "$1" ]
}

# Check if number
is_number() {
    [[ "$1" =~ ^-?[0-9]+$ ]]
}
```

---

## Project Structure

### Small Project Structure

```
project/
├── main.sh              # Entry point
├── lib/
│   ├── utils.sh         # Utility functions
│   └── config.sh        # Configuration
└── README.md
```

### Medium Project Structure

```
project/
├── main.sh
├── bin/
│   ├── deploy.sh
│   └── rollback.sh
├── lib/
│   ├── core/
│   │   ├── init.sh
│   │   └── cleanup.sh
│   ├── utils/
│   │   ├── string.sh
│   │   ├── file.sh
│   │   └── network.sh
│   └── validators.sh
├── config/
│   ├── default.conf
│   └── production.conf
└── README.md
```

### Large Project Structure (Enterprise)

```
project/
├── main.sh                    # Main entry point
├── bin/                       # Executable scripts
│   ├── deploy.sh
│   ├── monitor.sh
│   └── backup.sh
├── lib/                       # Library modules
│   ├── core/
│   │   ├── bootstrap.sh       # Initialization
│   │   ├── logging.sh         # Logging utilities
│   │   └── error_handler.sh   # Error handling
│   ├── utils/
│   │   ├── string.sh          # String utilities
│   │   ├── array.sh           # Array utilities
│   │   ├── file.sh            # File operations
│   │   └── network.sh         # Network utilities
│   ├── validators/
│   │   ├── input.sh           # Input validation
│   │   └── system.sh          # System checks
│   └── integrations/
│       ├── aws.sh             # AWS integration
│       ├── docker.sh          # Docker operations
│       └── slack.sh           # Slack notifications
├── config/
│   ├── default.conf           # Default configuration
│   ├── development.conf       # Dev environment
│   └── production.conf        # Prod environment
├── tests/
│   ├── test_utils.sh
│   └── test_validators.sh
├── docs/
│   └── API.md
└── README.md
```

---

## Import Patterns

### Pattern 1: Direct Import

```bash
#!/bin/bash
# Simple import for small projects

source lib/utils.sh
source lib/validators.sh

main() {
    validate_email "$1"
    process_data "$2"
}
```

### Pattern 2: Relative Path Import

```bash
#!/bin/bash
# Use script directory for imports

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

source "$SCRIPT_DIR/lib/config.sh"
source "$SCRIPT_DIR/lib/utils.sh"
```

### Pattern 3: Safe Import with Error Handling

```bash
#!/bin/bash

# Function to safely import modules
import() {
    local module="$1"
    local module_path="lib/${module}.sh"
    
    if [ ! -f "$module_path" ]; then
        echo "Error: Module '$module' not found at $module_path" >&2
        return 1
    fi
    
    if ! source "$module_path"; then
        echo "Error: Failed to load module '$module'" >&2
        return 1
    fi
    
    echo "✓ Loaded module: $module" >&2
    return 0
}

# Import required modules
import "validators" || exit 1
import "utils" || exit 1
import "network" || exit 1
```

### Pattern 4: Lazy Loading

```bash
#!/bin/bash
# Load modules only when needed

# Track loaded modules
declare -A LOADED_MODULES

load_module() {
    local module="$1"
    
    # Check if already loaded
    if [ "${LOADED_MODULES[$module]}" = "1" ]; then
        return 0
    fi
    
    # Load module
    if source "lib/${module}.sh" 2>/dev/null; then
        LOADED_MODULES[$module]=1
        echo "Loaded: $module" >&2
        return 0
    else
        echo "Failed to load: $module" >&2
        return 1
    fi
}

# Use lazy loading
process_network() {
    load_module "network"
    # Use network functions...
}

process_database() {
    load_module "database"
    # Use database functions...
}
```

### Pattern 5: Module Auto-Discovery

```bash
#!/bin/bash
# Automatically load all modules from directory

load_all_modules() {
    local lib_dir="${1:-lib}"
    
    if [ ! -d "$lib_dir" ]; then
        echo "Error: Library directory not found: $lib_dir" >&2
        return 1
    fi
    
    for module in "$lib_dir"/*.sh; do
        if [ -f "$module" ]; then
            echo "Loading: $(basename "$module")"
            source "$module" || {
                echo "Failed to load: $module" >&2
                return 1
            }
        fi
    done
}

load_all_modules "lib"
```

---

## Configuration Management

### Simple Configuration File

```bash
# config/app.conf
APP_NAME="MyApp"
APP_VERSION="1.0.0"
LOG_LEVEL="INFO"
LOG_FILE="/var/log/myapp.log"
MAX_RETRIES=3
TIMEOUT=30
```

### Loading Configuration

```bash
# lib/config.sh
#!/bin/bash

# Default configuration
APP_NAME="DefaultApp"
APP_VERSION="0.0.0"
LOG_LEVEL="DEBUG"

# Load configuration file
load_config() {
    local config_file="${1:-config/default.conf}"
    
    if [ ! -f "$config_file" ]; then
        echo "Warning: Config file not found: $config_file" >&2
        echo "Using defaults" >&2
        return 1
    fi
    
    # Source the configuration
    source "$config_file"
    echo "✓ Configuration loaded from: $config_file" >&2
}

# Show current configuration
show_config() {
    echo "=== Configuration ==="
    echo "App Name: $APP_NAME"
    echo "Version: $APP_VERSION"
    echo "Log Level: $LOG_LEVEL"
    echo "Log File: $LOG_FILE"
}
```

### Environment-Specific Configuration

```bash
#!/bin/bash

# Get environment (default to development)
ENV="${1:-development}"

# Load base configuration
source config/default.conf

# Override with environment-specific config
case "$ENV" in
    development)
        source config/development.conf
        ;;
    staging)
        source config/staging.conf
        ;;
    production)
        source config/production.conf
        ;;
    *)
        echo "Unknown environment: $ENV" >&2
        exit 1
        ;;
esac

echo "Running in $ENV mode"
```

### Secure Configuration (with secrets)

```bash
# lib/config.sh
#!/bin/bash

load_config() {
    local env="${1:-development}"
    
    # Load regular config
    [ -f "config/${env}.conf" ] && source "config/${env}.conf"
    
    # Load secrets from separate file (git-ignored)
    local secrets_file="config/.secrets.${env}"
    if [ -f "$secrets_file" ]; then
        # Set restrictive permissions
        chmod 600 "$secrets_file"
        source "$secrets_file"
    else
        echo "Warning: Secrets file not found" >&2
    fi
}

# config/.secrets.production (not in git)
DB_PASSWORD="secret_password"
API_KEY="secret_api_key"
AWS_SECRET="secret_aws_key"
```

---

## Error Handling Across Modules

### Centralized Error Handler

```bash
# lib/error_handler.sh
#!/bin/bash

# Error codes
readonly E_SUCCESS=0
readonly E_GENERAL=1
readonly E_INVALID_ARG=2
readonly E_FILE_NOT_FOUND=3
readonly E_PERMISSION_DENIED=4
readonly E_NETWORK_ERROR=5

# Error handler function
handle_error() {
    local exit_code=$1
    local message="$2"
    local line_number=${3:-"unknown"}
    
    echo "ERROR [$exit_code] at line $line_number: $message" >&2
    
    # Log to file if configured
    if [ -n "$ERROR_LOG" ]; then
        echo "$(date '+%Y-%m-%d %H:%M:%S') ERROR [$exit_code]: $message" >> "$ERROR_LOG"
    fi
    
    return $exit_code
}

# Trap errors
set -E
trap 'handle_error $? "Unexpected error" $LINENO' ERR
```

### Using Error Handler

```bash
#!/bin/bash

source lib/error_handler.sh

process_file() {
    local file="$1"
    
    if [ ! -f "$file" ]; then
        handle_error $E_FILE_NOT_FOUND "File not found: $file" $LINENO
        return $E_FILE_NOT_FOUND
    fi
    
    if [ ! -r "$file" ]; then
        handle_error $E_PERMISSION_DENIED "Cannot read file: $file" $LINENO
        return $E_PERMISSION_DENIED
    fi
    
    # Process file...
    return $E_SUCCESS
}
```

---

## Best Practices

### 1. Use Consistent Naming

```bash
# ✅ GOOD: Clear, consistent naming
lib/
├── string_utils.sh
├── file_utils.sh
├── network_utils.sh
└── validators.sh

# ❌ BAD: Inconsistent naming
lib/
├── str.sh
├── fileOps.sh
├── net-utils.sh
└── validate_stuff.sh
```

### 2. Document Modules

```bash
# lib/network.sh
#!/bin/bash

##############################################################################
# Network Utility Functions
# 
# This module provides network-related utilities including:
# - Port checking
# - Connectivity tests
# - URL validation
#
# Dependencies: curl, nc (netcat)
# Author: Your Name
# Version: 1.0.0
##############################################################################

# Check if port is open
# Arguments:
#   $1 - hostname/IP
#   $2 - port number
# Returns:
#   0 if port is open, 1 otherwise
is_port_open() {
    local host="$1"
    local port="$2"
    nc -z -w3 "$host" "$port" &>/dev/null
}
```

### 3. Avoid Circular Dependencies

```bash
# ❌ BAD: Circular dependency
# module_a.sh sources module_b.sh
# module_b.sh sources module_a.sh
# This creates an infinite loop!

# ✅ GOOD: Hierarchical dependencies
# main.sh
#   ├── sources core.sh
#   ├── sources utils.sh (may source core.sh)
#   └── sources app.sh (may source utils.sh and core.sh)
```

### 4. Use Include Guards

```bash
# lib/utils.sh
#!/bin/bash

# Include guard to prevent multiple sourcing
if [ -n "$UTILS_SH_LOADED" ]; then
    return 0
fi
readonly UTILS_SH_LOADED=1

# Module content...
function_one() { ... }
function_two() { ... }
```

### 5. Separate Concerns

```bash
# ✅ GOOD: Each module has single responsibility
lib/
├── logging.sh       # Only logging functions
├── validation.sh    # Only validation functions
├── database.sh      # Only database operations
└── email.sh         # Only email functions

# ❌ BAD: Mixed responsibilities
lib/
└── everything.sh    # All functions in one file
```

---

## Complete Example: Modular Backup System

### Project Structure

```
backup-system/
├── backup.sh           # Main script
├── lib/
│   ├── core.sh         # Core functions
│   ├── validators.sh   # Validation
│   ├── notifications.sh # Email/Slack notifications
│   └── logging.sh      # Logging utilities
└── config/
    └── backup.conf     # Configuration
```

### lib/core.sh

```bash
#!/bin/bash
[ -n "$CORE_SH_LOADED" ] && return 0
readonly CORE_SH_LOADED=1

perform_backup() {
    local source="$1"
    local dest="$2"
    local timestamp=$(date +%Y%m%d_%H%M%S)
    local backup_file="$dest/backup_${timestamp}.tar.gz"
    
    log_info "Starting backup: $source -> $backup_file"
    
    if tar -czf "$backup_file" "$source" 2>/dev/null; then
        log_success "Backup created: $backup_file"
        return 0
    else
        log_error "Backup failed"
        return 1
    fi
}

cleanup_old_backups() {
    local backup_dir="$1"
    local keep_days="${2:-7}"
    
    log_info "Cleaning up backups older than $keep_days days"
    find "$backup_dir" -name "backup_*.tar.gz" -mtime +$keep_days -delete
}
```

### lib/logging.sh

```bash
#!/bin/bash
[ -n "$LOGGING_SH_LOADED" ] && return 0
readonly LOGGING_SH_LOADED=1

LOG_FILE="${LOG_FILE:-/var/log/backup.log}"

log_message() {
    local level="$1"
    local message="$2"
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    
    echo "[$timestamp] [$level] $message" | tee -a "$LOG_FILE"
}

log_info() { log_message "INFO" "$1"; }
log_success() { log_message "SUCCESS" "$1"; }
log_warning() { log_message "WARNING" "$1"; }
log_error() { log_message "ERROR" "$1"; }
```

### lib/notifications.sh

```bash
#!/bin/bash
[ -n "$NOTIFICATIONS_SH_LOADED" ] && return 0
readonly NOTIFICATIONS_SH_LOADED=1

send_email() {
    local to="$1"
    local subject="$2"
    local body="$3"
    
    echo "$body" | mail -s "$subject" "$to"
}

send_slack() {
    local webhook="$1"
    local message="$2"
    
    curl -X POST -H 'Content-type: application/json' \
        --data "{\"text\":\"$message\"}" \
        "$webhook"
}
```

### backup.sh (Main Script)

```bash
#!/bin/bash

# Get script directory
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

# Load modules
source "$SCRIPT_DIR/lib/logging.sh"
source "$SCRIPT_DIR/lib/core.sh"
source "$SCRIPT_DIR/lib/notifications.sh"
source "$SCRIPT_DIR/config/backup.conf"

main() {
    log_info "Backup process started"
    
    # Perform backup
    if perform_backup "$SOURCE_DIR" "$BACKUP_DIR"; then
        cleanup_old_backups "$BACKUP_DIR" "$KEEP_DAYS"
        send_email "$ADMIN_EMAIL" "Backup Success" "Backup completed successfully"
        log_success "Backup process completed"
        return 0
    else
        send_email "$ADMIN_EMAIL" "Backup Failed" "Backup process failed"
        log_error "Backup process failed"
        return 1
    fi
}

main "$@"
```

---

## Summary

In this chapter, you learned:

- ✅ **Modular Design**: Benefits of breaking scripts into reusable modules
- ✅ **Source Command**: How to import and use external scripts
- ✅ **Libraries**: Creating function libraries for common operations
- ✅ **Project Structure**: Organizing small to enterprise-scale projects
- ✅ **Import Patterns**: Different strategies for loading modules
- ✅ **Configuration**: Managing settings across environments
- ✅ **Error Handling**: Centralized error management across modules

### Key Takeaways

| Concept | Best Practice |
|---------|---------------|
| Sourcing | Use relative paths from script directory |
| Libraries | One responsibility per module |
| Naming | Consistent, descriptive names |
| Documentation | Document dependencies and usage |
| Guards | Prevent multiple sourcing |
| Structure | Scale structure with project complexity |

### Next Steps

In [Chapter 14: Working with Files and Directories](../part-05-intermediate/14-files-and-directories.md), you'll learn advanced file operations, testing operators, and automation techniques for file management.

---

## Practice Exercises

1. **Exercise 1**: Create a modular calculator with separate modules for basic ops, scientific ops, and conversions
2. **Exercise 2**: Build a system monitoring toolkit with modules for CPU, memory, disk, and network
3. **Exercise 3**: Design a user management system with modules for creation, deletion, and reporting
4. **Exercise 4**: Create a configuration management system that supports multiple environments
5. **Exercise 5**: Build a logging framework with different log levels and output destinations
