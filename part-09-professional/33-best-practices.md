# Chapter 31: Best Practices for Writing Scripts

## Introduction

Professional Bash scripts are maintainable, readable, and robust. This chapter covers coding standards, documentation, and best practices.

---

## Script Structure

### Standard Script Template

```bash
#!/bin/bash
#
# Script Name: backup-system.sh
# Description: Automated system backup with rotation
# Author: Your Name
# Date: 2025-01-01
# Version: 1.0.0
#
# Usage: ./backup-system.sh [options]
# Options:
#   -d, --destination  Backup destination directory
#   -r, --retention    Number of backups to keep (default: 7)
#   -h, --help         Show this help message
#
# Examples:
#   ./backup-system.sh --destination /backup --retention 14
#

set -euo pipefail
IFS=$'\n\t'

# Constants
readonly SCRIPT_NAME=$(basename "$0")
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_VERSION="1.0.0"

# Default values
DEST_DIR="/backup"
RETENTION_DAYS=7
LOG_FILE="/var/log/backup.log"

# Functions
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

error_exit() {
    log "ERROR: $1"
    exit "${2:-1}"
}

show_help() {
    sed -n '/^# Usage:/,/^$/p' "$0" | sed 's/^# \?//'
    exit 0
}

# Main logic
main() {
    log "Starting backup..."
    # Your code here
    log "Backup completed"
}

# Parse arguments
while [[ $# -gt 0 ]]; do
    case $1 in
        -d|--destination)
            DEST_DIR="$2"
            shift 2
            ;;
        -r|--retention)
            RETENTION_DAYS="$2"
            shift 2
            ;;
        -h|--help)
            show_help
            ;;
        *)
            error_exit "Unknown option: $1"
            ;;
    esac
done

# Run main
main "$@"
```

---

## Naming Conventions

### Variables

```bash
# Constants: UPPERCASE with underscores
readonly MAX_RETRIES=3
readonly DB_HOST="localhost"

# Local variables: lowercase with underscores
user_name="john"
file_path="/tmp/data.txt"

# Environment variables: UPPERCASE
export PATH="/usr/local/bin:$PATH"
export DATABASE_URL="postgres://localhost/mydb"

# Function-local variables: use 'local'
my_function() {
    local temp_file=$(mktemp)
    local counter=0
}
```

### Functions

```bash
# Use descriptive names with underscores
create_backup() {
    # ...
}

validate_input() {
    # ...
}

send_notification() {
    # ...
}

# Prefix private functions with underscore
_internal_helper() {
    # ...
}
```

### Files

```bash
# Use kebab-case for scripts
backup-database.sh
deploy-application.sh
check-health.sh

# Use .sh extension
# Use descriptive names that indicate purpose
```

---

## Error Handling

### Comprehensive Error Handling

```bash
#!/bin/bash

set -euo pipefail

# Exit codes
readonly EXIT_SUCCESS=0
readonly EXIT_ERROR=1
readonly EXIT_INVALID_ARGS=2
readonly EXIT_FILE_NOT_FOUND=3

error_exit() {
    local message=$1
    local code=${2:-$EXIT_ERROR}
    
    echo "ERROR: $message" >&2
    
    # Cleanup if needed
    cleanup
    
    exit "$code"
}

cleanup() {
    # Remove temporary files
    rm -f /tmp/temp_file_$$
    
    # Release locks
    rm -f /var/run/script.lock
}

trap cleanup EXIT
trap 'error_exit "Script interrupted" 130' INT TERM

# Validate inputs
if [ $# -lt 1 ]; then
    error_exit "Missing required argument" $EXIT_INVALID_ARGS
fi

if [ ! -f "$1" ]; then
    error_exit "File not found: $1" $EXIT_FILE_NOT_FOUND
fi
```

---

## Input Validation

### Robust Input Validation

```bash
#!/bin/bash

validate_email() {
    local email=$1
    local regex="^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"
    
    if [[ ! $email =~ $regex ]]; then
        return 1
    fi
    return 0
}

validate_ip() {
    local ip=$1
    local regex="^([0-9]{1,3}\.){3}[0-9]{1,3}$"
    
    if [[ ! $ip =~ $regex ]]; then
        return 1
    fi
    
    # Check each octet
    IFS='.' read -ra OCTETS <<< "$ip"
    for octet in "${OCTETS[@]}"; do
        if [ "$octet" -gt 255 ]; then
            return 1
        fi
    done
    
    return 0
}

validate_number() {
    local num=$1
    local min=${2:-0}
    local max=${3:-100}
    
    if ! [[ "$num" =~ ^[0-9]+$ ]]; then
        return 1
    fi
    
    if [ "$num" -lt "$min" ] || [ "$num" -gt "$max" ]; then
        return 1
    fi
    
    return 0
}

# Usage
if ! validate_email "user@example.com"; then
    echo "Invalid email"
    exit 1
fi
```

---

## Documentation

### Inline Comments

```bash
#!/bin/bash

# GOOD: Explains why, not what
# Skip processing if file was modified in last 24 hours
if [ $(find "$file" -mtime -1) ]; then
    return 0
fi

# BAD: States the obvious
# Check if file exists
if [ -f "$file" ]; then
    # Open file
    cat "$file"
fi

# Document complex logic
# Calculate compound interest: P(1 + r/n)^(nt)
# P = principal, r = rate, n = compounds per year, t = years
calculate_interest() {
    local principal=$1
    local rate=$2
    local years=$3
    # Formula implementation...
}
```

### Function Documentation

```bash
#!/bin/bash

#######################################
# Backup database to specified location
# Globals:
#   DB_HOST
#   DB_USER
#   DB_PASS
# Arguments:
#   Database name
#   Backup destination directory
# Returns:
#   0 if successful, 1 on error
# Outputs:
#   Writes backup file path to stdout
#######################################
backup_database() {
    local db_name=$1
    local dest_dir=$2
    local backup_file="${dest_dir}/${db_name}_$(date +%Y%m%d).sql.gz"
    
    mysqldump -h "$DB_HOST" -u "$DB_USER" -p"$DB_PASS" "$db_name" | \
        gzip > "$backup_file"
    
    echo "$backup_file"
}
```

---

## Code Organization

### Modular Structure

```bash
# lib/logging.sh
log_info() {
    echo "[INFO] $*" | tee -a "$LOG_FILE"
}

log_error() {
    echo "[ERROR] $*" | tee -a "$LOG_FILE" >&2
}

# lib/validation.sh
validate_file() {
    local file=$1
    [ -f "$file" ] && [ -r "$file" ]
}

# main.sh
#!/bin/bash
source "$(dirname "$0")/lib/logging.sh"
source "$(dirname "$0")/lib/validation.sh"

log_info "Starting script"
if validate_file "/etc/config"; then
    log_info "Config file is valid"
fi
```

---

## Performance Considerations

### Efficient Coding Patterns

```bash
# SLOW: Calling external command in loop
for file in *.txt; do
    line_count=$(wc -l < "$file")
    echo "$file: $line_count"
done

# FAST: Process in single pipeline
wc -l *.txt

# SLOW: Multiple grep calls
grep "pattern1" file.txt > /tmp/p1
grep "pattern2" /tmp/p1 > /tmp/p2
grep "pattern3" /tmp/p2

# FAST: Single awk call
awk '/pattern1/ && /pattern2/ && /pattern3/' file.txt

# SLOW: Reading file line by line
while IFS= read -r line; do
    process "$line"
done < large_file.txt

# FAST: Process in chunks if possible
xargs -n 1000 process < large_file.txt
```

---

## Security Best Practices

### Secure Coding

```bash
#!/bin/bash

# Use absolute paths
PATH="/usr/local/bin:/usr/bin:/bin"

# Set restrictive umask
umask 0077

# Use mktemp for temporary files
temp_file=$(mktemp)
trap "rm -f '$temp_file'" EXIT

# Quote all variables
filename="$1"
grep "pattern" "$filename"  # Not: grep "pattern" $filename

# Validate input
if [[ ! "$username" =~ ^[a-z0-9_-]+$ ]]; then
    echo "Invalid username"
    exit 1
fi

# Don't expose sensitive data
# WRONG: echo "Password: $password"
# RIGHT: Log only non-sensitive info

# Use arrays for command arguments
files=("file 1.txt" "file 2.txt")
ls -l "${files[@]}"  # Properly quoted
```

---

## Testing

### Test Framework

```bash
#!/bin/bash
# test-functions.sh

source ./functions.sh

TESTS_PASSED=0
TESTS_FAILED=0

assert_equals() {
    local expected=$1
    local actual=$2
    local message=${3:-""}
    
    if [ "$expected" = "$actual" ]; then
        echo "✓ $message"
        ((TESTS_PASSED++))
    else
        echo "✗ $message"
        echo "  Expected: $expected"
        echo "  Actual: $actual"
        ((TESTS_FAILED++))
    fi
}

# Test cases
test_validate_email() {
    assert_equals "0" "$(validate_email 'user@example.com'; echo $?)" \
        "Valid email should pass"
    
    assert_equals "1" "$(validate_email 'invalid-email'; echo $?)" \
        "Invalid email should fail"
}

# Run tests
test_validate_email

echo ""
echo "Tests passed: $TESTS_PASSED"
echo "Tests failed: $TESTS_FAILED"

[ $TESTS_FAILED -eq 0 ]
```

---

## Best Practices Checklist

- [ ] Use `set -euo pipefail` at script start
- [ ] Include descriptive header comments
- [ ] Use meaningful variable and function names
- [ ] Validate all input parameters
- [ ] Implement comprehensive error handling
- [ ] Quote all variable expansions
- [ ] Use `readonly` for constants
- [ ] Use `local` for function variables
- [ ] Implement cleanup with `trap`
- [ ] Log important actions
- [ ] Document complex logic
- [ ] Test edge cases
- [ ] Use ShellCheck for linting

---

## Summary

- Follow consistent naming conventions
- Implement robust error handling
- Validate all inputs
- Document code thoroughly
- Write modular, reusable code
- Consider performance
- Follow security best practices
- Test your scripts

---

**Next Chapter:** [32 - Optimizing Bash Performance](32-optimizing-performance.md)
