# Chapter 20: Debugging Bash Scripts

## Introduction

Debugging is an essential skill for writing reliable Bash scripts. This chapter covers debugging techniques, tools, and strategies to identify and fix issues quickly.

---

## Basic Debugging Techniques

### Using echo for Debugging

The simplest debugging method:

```bash
#!/bin/bash

echo "DEBUG: Starting script"

USERNAME="john"
echo "DEBUG: USERNAME=$USERNAME"

if [ -z "$USERNAME" ]; then
    echo "DEBUG: Username is empty"
else
    echo "DEBUG: Username is set to: $USERNAME"
fi
```

### Debugging with Comments

```bash
#!/bin/bash

# DEBUG: Check what this variable contains
echo "PATH=$PATH"

# DEBUG: Verify file existence
ls -la /tmp/myfile.txt

# Your actual code
# process_file /tmp/myfile.txt
```

---

## Using bash -x (Trace Mode)

### Running Scripts in Trace Mode

```bash
# Run script with trace
bash -x myscript.sh

# Example output:
# + USERNAME=john
# + '[' -z john ']'
# + echo 'Username: john'
# Username: john
```

### Enabling Trace in Script

```bash
#!/bin/bash -x

# Everything below is traced automatically
USERNAME="john"
echo "Hello, $USERNAME"
```

### Selective Tracing with set -x

```bash
#!/bin/bash

echo "Normal execution"

set -x  # Enable tracing
complicated_variable="$HOME/$(date +%Y)/backup"
mkdir -p "$complicated_variable"
set +x  # Disable tracing

echo "Back to normal"
```

---

## Advanced Debugging with set Options

### Commonly Used Options

```bash
#!/bin/bash

set -x  # Print commands and arguments as executed
set -v  # Print shell input lines as read
set -e  # Exit on error
set -u  # Exit on undefined variable
set -o pipefail  # Pipeline returns status of last failed command

# Combine multiple options
set -xeuo pipefail
```

### Conditional Debugging

```bash
#!/bin/bash

# Enable debug mode via environment variable
if [ "${DEBUG:-}" = "1" ]; then
    set -x
fi

# Now run with: DEBUG=1 ./script.sh
```

---

## Using PS4 for Better Trace Output

The `PS4` variable controls the trace prompt:

```bash
#!/bin/bash

# Default PS4 is '+ '
# Customize for more information

export PS4='+(${BASH_SOURCE}:${LINENO}): ${FUNCNAME[0]:+${FUNCNAME[0]}(): }'

set -x

function test_function() {
    local var="test"
    echo "$var"
}

test_function

# Output:
# +(script.sh:12): test_function(): local var=test
# +(script.sh:13): test_function(): echo test
# test
```

### Enhanced PS4 with Colors

```bash
#!/bin/bash

# Add colors to trace output
export PS4=$'\033[0;33m+(${BASH_SOURCE}:${LINENO}):\033[0m ${FUNCNAME[0]:+${FUNCNAME[0]}(): }'

set -x

echo "This will have colored trace output"
```

---

## Logging for Debugging

### Creating Debug Logs

```bash
#!/bin/bash

DEBUG_LOG="/tmp/debug_$(date +%Y%m%d_%H%M%S).log"

debug() {
    if [ "${DEBUG:-0}" = "1" ]; then
        echo "[DEBUG $(date +'%T')] $*" | tee -a "$DEBUG_LOG" >&2
    fi
}

debug "Script started"
USERNAME="john"
debug "Username set to: $USERNAME"

# Run with: DEBUG=1 ./script.sh
```

### Function Call Tracing

```bash
#!/bin/bash

trace() {
    echo "TRACE: ${FUNCNAME[1]}() called from ${BASH_SOURCE[1]}:${BASH_LINENO[0]}" >&2
}

function outer() {
    trace
    inner
}

function inner() {
    trace
    echo "Inner function"
}

outer
```

---

## Debugging Functions and Variables

### Inspecting Variables

```bash
#!/bin/bash

# Print all variables
set | grep "^MY_"

# Print specific variable with type
declare -p MY_VARIABLE

# Print all functions
declare -F

# Print function definition
declare -f my_function
```

### Variable Debugging Helper

```bash
#!/bin/bash

debug_var() {
    local var_name=$1
    local var_value="${!var_name}"
    echo "DEBUG: $var_name='$var_value' (type: $(declare -p $var_name 2>/dev/null | awk '{print $2}'))" >&2
}

USERNAME="john"
declare -i COUNT=42
declare -a FRUITS=("apple" "banana")

debug_var USERNAME
debug_var COUNT
debug_var FRUITS
```

---

## Debugging with trap

### Tracking Script Execution

```bash
#!/bin/bash

# Trace every command
trap 'echo ">>> Executing: $BASH_COMMAND"' DEBUG

USERNAME="john"
echo "Hello, $USERNAME"
ls /tmp

# Output shows each command before execution
```

### Exit Debugging

```bash
#!/bin/bash

trap 'echo "Exit code: $? at line $LINENO"' EXIT

function risky_operation() {
    # Might fail
    ls /nonexistent 2>/dev/null
    return $?
}

risky_operation
echo "After risky operation"
```

### Error Line Reporting

```bash
#!/bin/bash
set -e

trap 'echo "Error at line $LINENO: $BASH_COMMAND"' ERR

echo "Starting..."
false  # This will trigger the trap
echo "This won't execute"
```

---

## Debugging Pipelines

### Finding Where Pipeline Fails

```bash
#!/bin/bash
set -o pipefail

# Debug each stage
cat file.txt | \
    tee >(echo "After cat: $(wc -l)" >&2) | \
    grep "pattern" | \
    tee >(echo "After grep: $(wc -l)" >&2) | \
    sort
```

### Pipeline with Error Checking

```bash
#!/bin/bash

debug_pipeline() {
    local stage=$1
    shift
    
    if ! "$@"; then
        echo "Pipeline failed at stage: $stage" >&2
        return 1
    fi
}

cat file.txt | \
    debug_pipeline "cat" cat | \
    debug_pipeline "grep" grep "pattern" | \
    debug_pipeline "sort" sort
```

---

## Using ShellCheck

ShellCheck is a static analysis tool for shell scripts.

### Installation

```bash
# Ubuntu/Debian
sudo apt install shellcheck

# macOS
brew install shellcheck

# Windows (WSL)
sudo apt install shellcheck
```

### Basic Usage

```bash
# Check a script
shellcheck myscript.sh

# Check with specific shell
shellcheck -s bash myscript.sh

# Exclude specific warnings
shellcheck -e SC2086 myscript.sh
```

### Common ShellCheck Warnings

```bash
#!/bin/bash

# SC2086: Quote variables to prevent word splitting
FILES="*.txt"
rm $FILES  # Warning: should be "$FILES"

# SC2068: Quote array expansions
ARGS=("arg1" "arg2")
command ${ARGS[@]}  # Should be "${ARGS[@]}"

# SC2155: Declare and assign separately
export VAR=$(command)  # Should be on separate lines

# SC2164: Use cd ... || exit
cd /some/dir  # Should be: cd /some/dir || exit
```

---

## Creating Debug Utilities

### Debug Function Library

```bash
#!/bin/bash

# Source this in your scripts for debugging utilities

# Debug levels
declare -g DEBUG_LEVEL=${DEBUG_LEVEL:-0}

debug() {
    [ "$DEBUG_LEVEL" -ge 1 ] && echo "DEBUG: $*" >&2
}

debug_verbose() {
    [ "$DEBUG_LEVEL" -ge 2 ] && echo "VERBOSE: $*" >&2
}

debug_trace() {
    [ "$DEBUG_LEVEL" -ge 3 ] && echo "TRACE: ${FUNCNAME[1]}(): $*" >&2
}

# Dump variable
dump_var() {
    local var_name=$1
    echo "=== $var_name ===" >&2
    declare -p "$var_name" 2>/dev/null >&2 || echo "Undefined" >&2
    echo >&2
}

# Stack trace
stack_trace() {
    echo "Stack trace:" >&2
    local frame=0
    while caller $frame; do
        ((frame++))
    done >&2
}
```

### Usage Example

```bash
#!/bin/bash
source ./debug_lib.sh

DEBUG_LEVEL=2

debug "Script started"
debug_verbose "Detailed information"

USERNAME="john"
dump_var USERNAME

function inner() {
    stack_trace
}

function outer() {
    inner
}

outer
```

---

## Debugging Common Issues

### Whitespace Issues

```bash
#!/bin/bash
set -x

# Show hidden characters
cat -A file.txt

# Debug whitespace in variables
VAR="  spaced  "
echo ">>$VAR<<"  # Shows spaces
```

### Quoting Issues

```bash
#!/bin/bash

# Debug quoting problems
FILES="file1.txt file2.txt"

echo "Unquoted:"
for file in $FILES; do
    echo "  - $file"
done

echo "Quoted:"
for file in "$FILES"; do
    echo "  - $file"
done
```

### Path Issues

```bash
#!/bin/bash

# Debug path problems
echo "Script location: ${BASH_SOURCE[0]}"
echo "Script directory: $(dirname "${BASH_SOURCE[0]}")"
echo "Current directory: $(pwd)"
echo "Real path: $(realpath "${BASH_SOURCE[0]}")"
```

---

## Interactive Debugging

### Using read for Breakpoints

```bash
#!/bin/bash

# Manual breakpoint
debug_break() {
    if [ "${DEBUG:-0}" = "1" ]; then
        echo "=== BREAKPOINT ===" >&2
        echo "Variables:" >&2
        set | grep "^[A-Z]" >&2
        read -p "Press Enter to continue..." >&2
    fi
}

USERNAME="john"
debug_break

echo "Processing $USERNAME"
debug_break
```

### Step-by-Step Execution

```bash
#!/bin/bash

step_debug() {
    local cmd="$BASH_COMMAND"
    read -p "Execute: $cmd ? [Y/n] " -n 1 -r
    echo
    if [[ $REPLY =~ ^[Nn]$ ]]; then
        return 1
    fi
}

if [ "${STEP_DEBUG:-0}" = "1" ]; then
    trap step_debug DEBUG
fi

# Run with: STEP_DEBUG=1 ./script.sh
```

---

## Best Practices

1. **Use set -x strategically** - only for complex sections
2. **Customize PS4** for meaningful trace output
3. **Implement log levels** (debug, info, warning, error)
4. **Use ShellCheck** before running scripts
5. **Add debug mode** controlled by environment variable
6. **Document debug options** in script headers
7. **Remove debug code** or make it conditional in production

---

## Practical Example: Fully Debuggable Script

```bash
#!/bin/bash
#
# Debuggable script template
#
# Usage:
#   DEBUG=1 ./script.sh          # Enable debug output
#   DEBUG=2 ./script.sh          # Enable verbose debug
#   TRACE=1 ./script.sh          # Enable execution trace
#

set -euo pipefail

# Debug configuration
DEBUG=${DEBUG:-0}
TRACE=${TRACE:-0}

[ "$TRACE" = "1" ] && set -x

# Enhanced trace prompt
export PS4='+(${BASH_SOURCE}:${LINENO}): ${FUNCNAME[0]:+${FUNCNAME[0]}(): }'

# Logging functions
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*"
}

debug() {
    [ "$DEBUG" -ge 1 ] && echo "[DEBUG] $*" >&2
}

debug_verbose() {
    [ "$DEBUG" -ge 2 ] && echo "[VERBOSE] $*" >&2
}

# Error handling
error_exit() {
    echo "[ERROR] $*" >&2
    exit 1
}

trap 'error_exit "Error at line $LINENO"' ERR

# Main script
main() {
    log "Script started"
    debug "Debug mode enabled (level: $DEBUG)"
    
    # Your logic here
    local username="john"
    debug_verbose "Processing user: $username"
    
    log "Script completed"
}

main "$@"
```

---

## Summary

- Use `bash -x` or `set -x` for execution tracing
- Customize `PS4` for better trace output
- Implement debug levels for flexible logging
- Use ShellCheck for static analysis
- Create reusable debug utilities
- Use `trap DEBUG` for step-by-step execution

---

## Practice Exercises

1. Add comprehensive debugging to an existing script
2. Create a debug library with different log levels
3. Write a script that can be debugged interactively with breakpoints
4. Debug a failing pipeline to identify which stage is failing

---

**Next Chapter:** [21 - Regular Expressions in Bash](21-regular-expressions.md)
