# Chapter 8: Command-Line Arguments

## Table of Contents
1. [Introduction to Arguments](#introduction-to-arguments)
2. [Positional Parameters](#positional-parameters)
3. [Special Parameters](#special-parameters)
4. [Parsing Arguments](#parsing-arguments)
5. [getopts for Option Parsing](#getopts-for-option-parsing)
6. [Long Options](#long-options)
7. [Advanced Techniques](#advanced-techniques)
8. [Best Practices](#best-practices)
9. [Summary](#summary)

---

## Introduction to Arguments

Command-line arguments allow users to pass data to scripts when they're executed.

```bash
# Script invocation
./script.sh arg1 arg2 arg3

# With options
./script.sh -a -b value -c

# Mixed
./script.sh --verbose input.txt output.txt
```

---

## Positional Parameters

### Basic Positional Parameters

```bash
#!/bin/bash
# script.sh

echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "Third argument: $3"

# Usage: ./script.sh apple banana cherry
# Output:
# Script name: ./script.sh
# First argument: apple
# Second argument: banana
# Third argument: cherry
```

### Accessing Arguments Beyond $9

```bash
#!/bin/bash

# Arguments 1-9 can be accessed directly
echo "$1 $2 $3 $4 $5 $6 $7 $8 $9"

# For argument 10+, use braces
echo "${10} ${11} ${12}"

# Or use shift
shift 9
echo "$1 $2 $3"  # Now these are arguments 10, 11, 12
```

### Script Name Variations

```bash
#!/bin/bash

# $0 contains full path as invoked
echo "Full: $0"

# Get just the script name
script_name=$(basename "$0")
echo "Name: $script_name"

# Get directory
script_dir=$(dirname "$0")
echo "Directory: $script_dir"

# Absolute path
script_path=$(realpath "$0")
echo "Absolute: $script_path"

# Examples:
# ./script.sh          -> $0 = ./script.sh
# bash script.sh       -> $0 = script.sh
# /full/path/script.sh -> $0 = /full/path/script.sh
```

---

## Special Parameters

### $# - Argument Count

```bash
#!/bin/bash

echo "Number of arguments: $#"

if [ $# -eq 0 ]; then
    echo "No arguments provided"
    exit 1
fi

if [ $# -lt 2 ]; then
    echo "Error: At least 2 arguments required"
    echo "Usage: $0 <input> <output>"
    exit 1
fi

echo "Correct number of arguments"
```

### $@ and $* - All Arguments

```bash
#!/bin/bash

echo "Using \$@:"
for arg in "$@"; do
    echo "  - $arg"
done

echo "Using \$*:"
for arg in "$*"; do
    echo "  - $arg"
done

# Run: ./script.sh "arg 1" "arg 2" "arg 3"

# "$@" output (preserves separate arguments):
#   - arg 1
#   - arg 2
#   - arg 3

# "$*" output (single string):
#   - arg 1 arg 2 arg 3
```

### Critical Difference: "$@" vs "$*"

```bash
#!/bin/bash

# Function to demonstrate difference
show_args() {
    echo "Function received $# arguments:"
    for arg in "$@"; do
        echo "  - [$arg]"
    done
}

# Test with "$@" (RECOMMENDED)
echo "Passing with \"\$@\":"
show_args "$@"

# Test with "$*"
echo "Passing with \"\$*\":"
show_args "$*"

# Run: ./script.sh "first arg" "second arg"

# With "$@":
# Function received 2 arguments:
#   - [first arg]
#   - [second arg]

# With "$*":
# Function received 1 argument:
#   - [first arg second arg]
```

### $? - Exit Status

```bash
#!/bin/bash

# Previous command's exit status
ls /tmp
echo "Exit status: $?"  # 0 (success)

ls /nonexistent 2>/dev/null
echo "Exit status: $?"  # Non-zero (failure)

# Use in conditions
if grep "pattern" file.txt > /dev/null 2>&1; then
    echo "Pattern found (exit status was $?)"
else
    echo "Pattern not found (exit status was $?)"
fi

# Custom exit codes
if [ ! -f "required.txt" ]; then
    echo "Error: required.txt not found"
    exit 1
fi

# Multiple status codes
case $? in
    0) echo "Success";;
    1) echo "General error";;
    2) echo "Misuse of command";;
    126) echo "Command cannot execute";;
    127) echo "Command not found";;
    *) echo "Unknown error: $?";;
esac
```

### $$ - Process ID

```bash
#!/bin/bash

echo "Script PID: $$"

# Create unique temporary file
temp_file="/tmp/script_$$.tmp"
echo "Temp file: $temp_file"

# Cleanup on exit
trap "rm -f $temp_file" EXIT

# Use in logging
log_file="/var/log/script_$$.log"
echo "$(date): Script started" >> "$log_file"
```

### $! - Last Background Process

```bash
#!/bin/bash

# Start background process
sleep 10 &
bg_pid=$!

echo "Background process PID: $bg_pid"
echo "Also available as: $!"

# Wait for background process
wait $bg_pid
echo "Background process finished"

# Monitor background process
long_task &
task_pid=$!

while kill -0 $task_pid 2>/dev/null; do
    echo "Task $task_pid still running..."
    sleep 1
done
echo "Task completed"
```

### $_ - Last Argument

```bash
#!/bin/bash

# Last argument of previous command
mkdir -p /tmp/test
cd $_  # Goes to /tmp/test

echo "one two three"
echo "Last argument was: $_"  # three

# Useful in scripts
cp important.txt important.txt.bak
echo "Backup created: $_"
```

---

## Parsing Arguments

### Simple Argument Parsing

```bash
#!/bin/bash

# Check argument count
if [ $# -ne 2 ]; then
    echo "Usage: $0 <source> <destination>"
    exit 1
fi

source="$1"
dest="$2"

# Validate arguments
if [ ! -f "$source" ]; then
    echo "Error: Source file not found: $source"
    exit 1
fi

echo "Copying $source to $dest"
cp "$source" "$dest"
```

### Manual Option Parsing

```bash
#!/bin/bash

# Parse options manually
verbose=false
output=""

while [ $# -gt 0 ]; do
    case "$1" in
        -v|--verbose)
            verbose=true
            shift
            ;;
        -o|--output)
            output="$2"
            shift 2
            ;;
        -h|--help)
            echo "Usage: $0 [-v] [-o output] file"
            exit 0
            ;;
        -*)
            echo "Unknown option: $1"
            exit 1
            ;;
        *)
            # Positional argument
            file="$1"
            shift
            ;;
    esac
done

# Use parsed values
[ "$verbose" = true ] && echo "Verbose mode enabled"
[ -n "$output" ] && echo "Output file: $output"
[ -n "$file" ] && echo "Processing: $file"
```

---

## getopts for Option Parsing

### Basic getopts Usage

```bash
#!/bin/bash

# Parse short options
while getopts "vho:" opt; do
    case $opt in
        v)
            verbose=true
            ;;
        h)
            echo "Usage: $0 [-v] [-o output] file"
            exit 0
            ;;
        o)
            output="$OPTARG"
            ;;
        \?)
            echo "Invalid option: -$OPTARG" >&2
            exit 1
            ;;
    esac
done

# Shift past parsed options
shift $((OPTIND-1))

# Remaining arguments
file="$1"

# Use values
[ "$verbose" = true ] && echo "Verbose enabled"
[ -n "$output" ] && echo "Output: $output"
[ -n "$file" ] && echo "File: $file"
```

### getopts with Required Arguments

```bash
#!/bin/bash

usage() {
    echo "Usage: $0 [-u user] [-p password] [-h host] [-P port]"
    exit 1
}

# Default values
host="localhost"
port=3306

# Parse options
while getopts "u:p:h:P:" opt; do
    case $opt in
        u) username="$OPTARG";;
        p) password="$OPTARG";;
        h) host="$OPTARG";;
        P) port="$OPTARG";;
        \?) usage;;
    esac
done

# Validate required options
if [ -z "$username" ] || [ -z "$password" ]; then
    echo "Error: Username and password required"
    usage
fi

echo "Connecting to $host:$port as $username"
```

### Complete getopts Example

```bash
#!/bin/bash
# backup.sh - Backup script with options

usage() {
    cat << EOF
Usage: $0 [OPTIONS] <source>

Backup utility script

OPTIONS:
    -d, --destination DIR   Backup destination (default: /backup)
    -c, --compress          Compress backup
    -v, --verbose           Verbose output
    -e, --exclude PATTERN   Exclude pattern (can be used multiple times)
    -h, --help              Show this help

EXAMPLES:
    $0 /home/user
    $0 -c -d /mnt/backup /home/user
    $0 -v -e "*.tmp" -e "*.log" /var/www
EOF
    exit 0
}

# Defaults
destination="/backup"
compress=false
verbose=false
exclude_patterns=()

# Parse options
while getopts "d:cve:h" opt; do
    case $opt in
        d) destination="$OPTARG";;
        c) compress=true;;
        v) verbose=true;;
        e) exclude_patterns+=("$OPTARG");;
        h) usage;;
        \?) echo "Invalid option: -$OPTARG" >&2; usage;;
        :) echo "Option -$OPTARG requires an argument" >&2; usage;;
    esac
done

shift $((OPTIND-1))

# Get source
source="${1:?Error: Source directory required}"

# Build command
cmd="rsync -a"
[ "$verbose" = true ] && cmd="$cmd -v"
for pattern in "${exclude_patterns[@]}"; do
    cmd="$cmd --exclude '$pattern'"
done
cmd="$cmd '$source' '$destination/'"

# Execute
[ "$verbose" = true ] && echo "Executing: $cmd"
eval "$cmd"

# Compress if requested
if [ "$compress" = true ]; then
    [ "$verbose" = true ] && echo "Compressing backup..."
    tar -czf "${destination}/backup_$(date +%Y%m%d).tar.gz" \
        -C "$destination" "$(basename "$source")"
fi

echo "Backup completed"
```

---

## Long Options

### Manual Long Option Parsing

```bash
#!/bin/bash

while [ $# -gt 0 ]; do
    case "$1" in
        --verbose|-v)
            verbose=true
            shift
            ;;
        --output|-o)
            output="$2"
            shift 2
            ;;
        --config|-c)
            config="$2"
            shift 2
            ;;
        --help|-h)
            show_help
            exit 0
            ;;
        --*)
            echo "Unknown option: $1" >&2
            exit 1
            ;;
        *)
            # Positional argument
            args+=("$1")
            shift
            ;;
    esac
done
```

### Using getopt (Enhanced)

```bash
#!/bin/bash

# getopt (not getopts) supports long options
# Install: sudo apt install util-linux

TEMP=$(getopt -o 'vho:' \
              --long 'verbose,help,output:' \
              -n "$0" -- "$@")

if [ $? -ne 0 ]; then
    echo 'Error parsing arguments' >&2
    exit 1
fi

eval set -- "$TEMP"
unset TEMP

# Parse options
while true; do
    case "$1" in
        '-v'|'--verbose')
            verbose=true
            shift
            ;;
        '-o'|'--output')
            output="$2"
            shift 2
            ;;
        '-h'|'--help')
            show_help
            exit 0
            ;;
        '--')
            shift
            break
            ;;
        *)
            echo "Internal error!" >&2
            exit 1
            ;;
    esac
done

# Remaining arguments
for arg in "$@"; do
    echo "Positional: $arg"
done
```

---

## Advanced Techniques

### Shift Command

```bash
#!/bin/bash

echo "Initial arguments: $@"
echo "Argument count: $#"

# Process and remove first argument
while [ $# -gt 0 ]; do
    echo "Processing: $1"
    shift  # Remove $1, shift all others down
    echo "Remaining: $@"
done

# Shift multiple positions
shift 3  # Remove first 3 arguments
```

### Default Values

```bash
#!/bin/bash

# Use default if argument not provided
input="${1:-input.txt}"
output="${2:-output.txt}"
mode="${3:-normal}"

echo "Input: $input"
echo "Output: $output"
echo "Mode: $mode"

# Alternative: parameter expansion
: ${VAR:=default}  # Set if unset or empty
: ${VAR=default}   # Set if unset only
```

### Validation Functions

```bash
#!/bin/bash

# Validate file exists
validate_file() {
    if [ ! -f "$1" ]; then
        echo "Error: File not found: $1" >&2
        return 1
    fi
    return 0
}

# Validate number
validate_number() {
    if ! [[ "$1" =~ ^[0-9]+$ ]]; then
        echo "Error: Not a valid number: $1" >&2
        return 1
    fi
    return 0
}

# Validate email
validate_email() {
    if ! [[ "$1" =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
        echo "Error: Invalid email: $1" >&2
        return 1
    fi
    return 0
}

# Use in script
if ! validate_file "$1"; then
    exit 1
fi

if ! validate_number "$2"; then
    exit 1
fi
```

### Flexible Argument Handling

```bash
#!/bin/bash

# Accept arguments in any order
files=()
verbose=false
output=""

for arg in "$@"; do
    case "$arg" in
        -v|--verbose)
            verbose=true
            ;;
        -o=*|--output=*)
            output="${arg#*=}"
            ;;
        -*)
            echo "Unknown option: $arg" >&2
            exit 1
            ;;
        *)
            files+=("$arg")
            ;;
    esac
done

# Process files
for file in "${files[@]}"; do
    [ "$verbose" = true ] && echo "Processing: $file"
done
```

---

## Best Practices

### 1. Always Validate Arguments

```bash
#!/bin/bash

if [ $# -ne 2 ]; then
    echo "Usage: $0 <source> <destination>" >&2
    exit 1
fi

if [ ! -f "$1" ]; then
    echo "Error: Source file not found: $1" >&2
    exit 1
fi
```

### 2. Provide Help Message

```bash
#!/bin/bash

show_help() {
    cat << EOF
Usage: $0 [OPTIONS] <file>

Description of what the script does

OPTIONS:
    -v, --verbose    Enable verbose output
    -o FILE          Output file
    -h, --help       Show this help

EXAMPLES:
    $0 input.txt
    $0 -v -o result.txt input.txt
EOF
}

[ "$1" = "-h" ] || [ "$1" = "--help" ] && show_help && exit 0
```

### 3. Use Meaningful Variable Names

```bash
# BAD
a="$1"
b="$2"

# GOOD
input_file="$1"
output_file="$2"
```

### 4. Quote All Variables

```bash
# Always quote to prevent word splitting
cp "$source" "$destination"
rm "$filename"
echo "File: $filepath"
```

### 5. Set Defaults

```bash
# Provide sensible defaults
config_file="${1:-/etc/app/config.conf}"
log_level="${LOG_LEVEL:-INFO}"
max_retries="${2:-3}"
```

### 6. Use Readonly for Constants

```bash
#!/bin/bash

readonly SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
readonly CONFIG_FILE="$SCRIPT_DIR/config.conf"
readonly VERSION="1.0.0"
```

---

## Summary

### Positional Parameters

```bash
$0          # Script name
$1-$9       # Arguments 1-9
${10}       # Arguments 10+
```

### Special Parameters

```bash
$#          # Argument count
$@          # All arguments (recommended)
$*          # All arguments (as single string)
$?          # Exit status
$$          # Process ID
$!          # Last background PID
$_          # Last argument
```

### Parsing Methods

```bash
# Manual parsing
case "$1" in
    -v) verbose=true;;
    *) file="$1";;
esac

# getopts (short options)
while getopts "vho:" opt; do
    case $opt in
        v) verbose=true;;
        o) output="$OPTARG";;
    esac
done

# getopt (long options)
getopt -o 'vh' --long 'verbose,help' -- "$@"
```

### Common Patterns

```bash
# Check argument count
[ $# -eq 0 ] && usage && exit 1

# Loop through arguments
for arg in "$@"; do
    echo "$arg"
done

# Shift arguments
while [ $# -gt 0 ]; do
    process "$1"
    shift
done

# Default values
file="${1:-default.txt}"

# Validation
[ ! -f "$1" ] && echo "File not found" && exit 1
```

### Complete Example Template

```bash
#!/bin/bash

set -euo pipefail

readonly SCRIPT_NAME="$(basename "$0")"
readonly VERSION="1.0.0"

usage() {
    cat << EOF
Usage: $SCRIPT_NAME [OPTIONS] <file>

OPTIONS:
    -v, --verbose       Verbose output
    -o, --output FILE   Output file
    -h, --help          Show help

Example: $SCRIPT_NAME -v input.txt
EOF
    exit 0
}

# Defaults
verbose=false
output=""

# Parse options
while getopts "vho:" opt; do
    case $opt in
        v) verbose=true;;
        o) output="$OPTARG";;
        h) usage;;
        \?) echo "Invalid option: -$OPTARG" >&2; exit 1;;
    esac
done

shift $((OPTIND-1))

# Get positional arguments
file="${1:?Error: File argument required}"

# Validate
[ ! -f "$file" ] && echo "Error: File not found: $file" >&2 && exit 1

# Execute
[ "$verbose" = true ] && echo "Processing $file..."
```

---

**Next Chapter:** [Conditional Statements →](../part-03-control-structures/09-conditional-statements.md)

---

*Practice Exercise: Create a script that accepts multiple command-line options using getopts, validates all inputs, provides a help message, and handles both short and long-form arguments.*
