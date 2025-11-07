# Chapter 17: Error Handling and Exit Codes

## Introduction

Robust error handling is what separates production-ready scripts from quick hacks. This chapter covers exit codes, error trapping, and best practices for handling failures gracefully.

---

## Understanding Exit Codes

Every command in Bash returns an exit code (0-255):
- **0** = Success
- **Non-zero** = Failure (specific meaning varies by command)

### Checking Exit Status

```bash
# $? holds the exit code of the last command
ls /some/path
echo $?  # 0 if successful, 2 if not found

# Using in conditions
if [ $? -eq 0 ]; then
    echo "Command succeeded"
fi
```

### Setting Exit Codes in Scripts

```bash
#!/bin/bash

function backup_file() {
    local source=$1
    local dest=$2
    
    if [ ! -f "$source" ]; then
        echo "Error: Source file not found" >&2
        return 1
    fi
    
    cp "$source" "$dest"
    return $?
}

# Exit the script with status
exit 0  # Success
exit 1  # General error
```

---

## Error Handling with set Options

### set -e: Exit on Error

```bash
#!/bin/bash
set -e  # Exit immediately if any command fails

mkdir /tmp/mydir
cd /tmp/mydir
touch file.txt

# If any command above fails, script stops
```

### set -u: Exit on Undefined Variables

```bash
#!/bin/bash
set -u  # Treat unset variables as errors

NAME="John"
echo "$NAME"      # OK
echo "$SURNAME"   # Error: unbound variable
```

### set -o pipefail: Catch Pipe Failures

```bash
#!/bin/bash
set -o pipefail  # Pipeline fails if any command fails

# Without pipefail, only the exit code of 'tail' matters
# With pipefail, if grep fails, whole pipeline fails
cat file.txt | grep "pattern" | tail -n 1
```

### Combining Options

```bash
#!/bin/bash
set -euo pipefail

# Strict mode: fail on errors, undefined vars, and pipe failures
```

---

## Using trap for Cleanup

`trap` executes commands when the script exits or receives signals.

### Basic trap Usage

```bash
#!/bin/bash

# Cleanup function
cleanup() {
    echo "Cleaning up temporary files..."
    rm -f /tmp/mytemp.$$
}

# Set trap to call cleanup on exit
trap cleanup EXIT

# Create temp file
touch /tmp/mytemp.$$
echo "Working..."

# cleanup() runs automatically on exit
```

### Trapping Errors

```bash
#!/bin/bash
set -e

error_handler() {
    echo "Error occurred in script at line $1" >&2
    exit 1
}

trap 'error_handler $LINENO' ERR

# Your commands here
ls /nonexistent/path  # Triggers error handler
```

### Trapping Signals

```bash
#!/bin/bash

# Handle Ctrl+C gracefully
trap 'echo "Interrupted!"; exit 130' INT

# Handle termination
trap 'echo "Terminated!"; cleanup; exit 143' TERM

while true; do
    echo "Running... (Press Ctrl+C to stop)"
    sleep 2
done
```

### Multiple Cleanup Tasks

```bash
#!/bin/bash

temp_file=$(mktemp)
temp_dir=$(mktemp -d)

cleanup() {
    echo "Removing temporary files..."
    rm -f "$temp_file"
    rm -rf "$temp_dir"
    # Disconnect from services
    # Unlock resources
}

trap cleanup EXIT

# Work with temporary resources
echo "Data" > "$temp_file"
```

---

## Custom Error Messages and Logging

### Redirect Errors to STDERR

```bash
#!/bin/bash

error() {
    echo "ERROR: $*" >&2
}

warning() {
    echo "WARNING: $*" >&2
}

info() {
    echo "INFO: $*"
}

# Usage
if [ ! -f "config.txt" ]; then
    error "Configuration file not found"
    exit 1
fi

info "Starting process..."
```

### Logging to File

```bash
#!/bin/bash

LOG_FILE="/var/log/myscript.log"

log() {
    local level=$1
    shift
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] [$level] $*" | tee -a "$LOG_FILE"
}

log INFO "Script started"
log ERROR "Something went wrong"
log WARNING "Deprecated function used"
```

### Combined Error Handling and Logging

```bash
#!/bin/bash

LOG_FILE="/var/log/backup.log"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

error_exit() {
    log "ERROR: $1"
    exit "${2:-1}"
}

cleanup() {
    log "Cleaning up..."
    # Cleanup tasks
}

trap cleanup EXIT
set -euo pipefail

# Main script
log "Starting backup..."

if [ ! -d "/backup/source" ]; then
    error_exit "Source directory not found" 2
fi

log "Backup completed successfully"
```

---

## Advanced Error Handling Patterns

### Try-Catch Pattern

```bash
#!/bin/bash

try() {
    "$@"
    return $?
}

catch() {
    return $?
}

# Usage
if ! try command_that_might_fail arg1 arg2; then
    catch
    echo "Command failed, handling error..."
fi
```

### Retry Logic

```bash
#!/bin/bash

retry() {
    local max_attempts=$1
    local delay=$2
    shift 2
    local command="$@"
    local attempt=1
    
    until $command; do
        if [ $attempt -ge $max_attempts ]; then
            echo "Command failed after $max_attempts attempts" >&2
            return 1
        fi
        echo "Attempt $attempt failed. Retrying in ${delay}s..." >&2
        sleep $delay
        ((attempt++))
    done
}

# Usage: retry 5 10 curl https://example.com
retry 3 2 ping -c 1 google.com
```

### Validation with Exit Codes

```bash
#!/bin/bash

validate_file() {
    local file=$1
    
    [ -z "$file" ] && { echo "No file specified" >&2; return 1; }
    [ ! -e "$file" ] && { echo "File doesn't exist: $file" >&2; return 2; }
    [ ! -f "$file" ] && { echo "Not a regular file: $file" >&2; return 3; }
    [ ! -r "$file" ] && { echo "File not readable: $file" >&2; return 4; }
    
    return 0
}

# Usage
if validate_file "/etc/passwd"; then
    echo "File is valid"
else
    exit_code=$?
    echo "Validation failed with code: $exit_code"
    exit $exit_code
fi
```

---

## Best Practices

1. **Always check exit codes** for critical operations
2. **Use `set -euo pipefail`** for strict error handling
3. **Implement cleanup with trap** for temporary resources
4. **Log to STDERR** for errors and warnings
5. **Provide meaningful exit codes** (use standard codes where applicable)
6. **Document your error codes** in script comments
7. **Test failure scenarios** explicitly

### Standard Exit Codes

```bash
# Common exit codes
EXIT_SUCCESS=0
EXIT_GENERAL_ERROR=1
EXIT_MISUSE=2
EXIT_CANNOT_EXECUTE=126
EXIT_NOT_FOUND=127
EXIT_INVALID_ARG=128
EXIT_CTRL_C=130
```

---

## Practical Example: Production-Ready Script

```bash
#!/bin/bash
#
# Production backup script with comprehensive error handling
#

set -euo pipefail

# Constants
readonly SCRIPT_NAME=$(basename "$0")
readonly LOG_FILE="/var/log/${SCRIPT_NAME}.log"
readonly LOCK_FILE="/var/run/${SCRIPT_NAME}.lock"

# Exit codes
readonly EXIT_SUCCESS=0
readonly EXIT_ERROR=1
readonly EXIT_ALREADY_RUNNING=2

# Logging
log() {
    local level=$1
    shift
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] [$level] $*" | tee -a "$LOG_FILE"
}

error_exit() {
    log ERROR "$1"
    exit "${2:-$EXIT_ERROR}"
}

# Cleanup
cleanup() {
    log INFO "Performing cleanup..."
    rm -f "$LOCK_FILE"
}

trap cleanup EXIT
trap 'error_exit "Script interrupted" 130' INT TERM

# Lock file to prevent concurrent execution
if [ -f "$LOCK_FILE" ]; then
    error_exit "Script already running (lock file exists)" $EXIT_ALREADY_RUNNING
fi

touch "$LOCK_FILE"

# Main logic
log INFO "Starting backup process..."

# Your backup logic here
# ...

log INFO "Backup completed successfully"
exit $EXIT_SUCCESS
```

---

## Summary

- Exit codes indicate success (0) or failure (non-zero)
- Use `set -e`, `set -u`, and `set -o pipefail` for strict error handling
- `trap` allows cleanup on exit or error
- Always log errors to STDERR
- Implement retry logic for unreliable operations
- Use meaningful exit codes and document them

---

## Practice Exercises

1. Write a script that validates all its dependencies and exits with specific codes for each missing tool
2. Create a backup script with retry logic and comprehensive logging
3. Implement a script that uses `trap` to clean up multiple temporary resources
4. Build an error handler that sends email notifications on critical failures

---

**Next Chapter:** [18 - Debugging Bash Scripts](18-debugging-bash-scripts.md)
