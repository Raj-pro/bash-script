# Chapter 12: Bash Functions

## Introduction

Functions are the cornerstone of modular and reusable Bash scripting. They allow you to encapsulate logic, reduce code duplication, and create maintainable scripts. In this chapter, you'll learn how to declare, call, and manage functions effectively, including scope management, return values, and parameter passing.

## Table of Contents
- [What Are Functions?](#what-are-functions)
- [Declaring Functions](#declaring-functions)
- [Calling Functions](#calling-functions)
- [Function Arguments](#function-arguments)
- [Return Values](#return-values)
- [Variable Scope](#variable-scope)
- [Recursive Functions](#recursive-functions)
- [Best Practices](#best-practices)

---

## What Are Functions?

A function is a named block of code that can be executed multiple times without rewriting the code. Functions improve:
- **Code Reusability**: Write once, use many times
- **Maintainability**: Update logic in one place
- **Readability**: Break complex scripts into logical chunks
- **Testing**: Easier to test individual components

### Why Use Functions?

```bash
# Without functions (repetitive)
echo "Starting backup..."
tar -czf backup1.tar.gz /data
echo "Backup complete"

echo "Starting backup..."
tar -czf backup2.tar.gz /logs
echo "Backup complete"

# With functions (clean and reusable)
backup() {
    echo "Starting backup..."
    tar -czf "$1" "$2"
    echo "Backup complete"
}

backup backup1.tar.gz /data
backup backup2.tar.gz /logs
```

---

## Declaring Functions

There are two primary syntaxes for declaring functions in Bash:

### Syntax 1: Function Keyword

```bash
function function_name {
    # function body
    commands
}
```

### Syntax 2: POSIX Style (Recommended)

```bash
function_name() {
    # function body
    commands
}
```

### Examples

```bash
# Simple greeting function
greet() {
    echo "Hello, World!"
}

# Function with commands
show_date() {
    echo "Current date and time:"
    date "+%Y-%m-%d %H:%M:%S"
}

# Multi-line function
system_info() {
    echo "=== System Information ==="
    echo "Hostname: $(hostname)"
    echo "Uptime: $(uptime -p)"
    echo "Kernel: $(uname -r)"
}
```

### Function Naming Conventions

```bash
# Use descriptive names
check_disk_space() { df -h; }
backup_database() { mysqldump...; }

# Use underscores for readability
create_user_account() { ... }
send_email_notification() { ... }

# Prefix with action verbs
get_cpu_usage() { top -bn1 | grep "Cpu(s)"; }
validate_input() { ... }
process_logs() { ... }
```

---

## Calling Functions

Once declared, functions can be called by their name:

```bash
#!/bin/bash

# Declare function
say_hello() {
    echo "Hello from function!"
}

# Call function
say_hello

# Call multiple times
say_hello
say_hello
```

### Calling Functions in Scripts

```bash
#!/bin/bash

# Define all functions first
show_menu() {
    echo "1. Option 1"
    echo "2. Option 2"
    echo "3. Exit"
}

process_choice() {
    case $1 in
        1) echo "Processing option 1";;
        2) echo "Processing option 2";;
        3) echo "Exiting..."; exit 0;;
    esac
}

# Main execution
show_menu
read -p "Enter choice: " choice
process_choice "$choice"
```

### Order Matters

```bash
# ❌ WRONG - Function called before declaration
greet
greet() { echo "Hello"; }

# ✅ CORRECT - Function declared before calling
greet() { echo "Hello"; }
greet
```

---

## Function Arguments

Functions can accept arguments similar to scripts:

### Positional Parameters

| Parameter | Description |
|-----------|-------------|
| `$0` | Script name (not function name) |
| `$1, $2, ...` | Function arguments |
| `$@` | All arguments as separate words |
| `$*` | All arguments as single string |
| `$#` | Number of arguments |

### Examples

```bash
# Function with single argument
greet_user() {
    echo "Hello, $1!"
}

greet_user "Alice"    # Output: Hello, Alice!
greet_user "Bob"      # Output: Hello, Bob!

# Function with multiple arguments
add_numbers() {
    local result=$(( $1 + $2 ))
    echo "Sum: $result"
}

add_numbers 10 20     # Output: Sum: 30

# Function using all arguments
show_args() {
    echo "Number of arguments: $#"
    echo "All arguments: $@"
    echo "First argument: $1"
    echo "Second argument: $2"
}

show_args apple banana cherry
# Output:
# Number of arguments: 3
# All arguments: apple banana cherry
# First argument: apple
# Second argument: banana
```

### Iterating Over Arguments

```bash
# Process all arguments
print_all() {
    echo "Processing $# arguments:"
    for arg in "$@"; do
        echo "  - $arg"
    done
}

print_all file1.txt file2.txt file3.txt
# Output:
# Processing 3 arguments:
#   - file1.txt
#   - file2.txt
#   - file3.txt
```

### Default Arguments

```bash
# Using default values
greet_with_default() {
    local name="${1:-Guest}"
    echo "Hello, $name!"
}

greet_with_default           # Output: Hello, Guest!
greet_with_default "Alice"   # Output: Hello, Alice!

# Multiple defaults
create_backup() {
    local source="${1:-/data}"
    local dest="${2:-/backup}"
    local date=$(date +%Y%m%d)
    
    echo "Backing up $source to $dest/backup-$date.tar.gz"
    tar -czf "$dest/backup-$date.tar.gz" "$source"
}

create_backup                      # Uses defaults
create_backup /var/log /backups   # Uses provided values
```

### Argument Validation

```bash
# Check argument count
backup_file() {
    if [ $# -ne 2 ]; then
        echo "Usage: backup_file <source> <destination>"
        return 1
    fi
    
    cp "$1" "$2"
    echo "Backed up $1 to $2"
}

# Check argument validity
divide() {
    if [ $# -ne 2 ]; then
        echo "Error: Two arguments required"
        return 1
    fi
    
    if [ "$2" -eq 0 ]; then
        echo "Error: Division by zero"
        return 1
    fi
    
    echo $(( $1 / $2 ))
}

divide 10 2   # Output: 5
divide 10 0   # Output: Error: Division by zero
```

---

## Return Values

Bash functions can return values in two ways:

### 1. Exit Status (return)

The `return` statement returns an exit code (0-255):

```bash
# Return success/failure
check_file() {
    if [ -f "$1" ]; then
        return 0    # Success
    else
        return 1    # Failure
    fi
}

# Use in conditionals
if check_file "config.txt"; then
    echo "File exists"
else
    echo "File not found"
fi
```

### 2. Output (echo/printf)

Use command substitution to capture output:

```bash
# Return string value
get_username() {
    echo "$USER"
}

current_user=$(get_username)
echo "Current user: $current_user"

# Return calculated value
multiply() {
    echo $(( $1 * $2 ))
}

result=$(multiply 5 7)
echo "5 × 7 = $result"

# Return multiple values
get_stats() {
    local count=10
    local total=100
    local avg=$(( total / count ))
    
    echo "$count $total $avg"
}

read count total avg <<< $(get_stats)
echo "Count: $count, Total: $total, Average: $avg"
```

### Return Value Patterns

```bash
# Pattern 1: Boolean functions (return 0 or 1)
is_root() {
    [ "$(id -u)" -eq 0 ]
}

if is_root; then
    echo "Running as root"
fi

# Pattern 2: Status code with message
validate_email() {
    if [[ $1 =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
        echo "Valid email"
        return 0
    else
        echo "Invalid email"
        return 1
    fi
}

result=$(validate_email "user@example.com")
status=$?
echo "$result (Exit code: $status)"

# Pattern 3: Output + exit code
process_data() {
    if [ -z "$1" ]; then
        echo "Error: No data provided" >&2
        return 1
    fi
    
    # Process and return result
    echo "Processed: $1"
    return 0
}
```

---

## Variable Scope

Understanding variable scope is crucial for writing bug-free functions:

### Global Variables

Variables declared outside functions are global:

```bash
#!/bin/bash

# Global variable
username="Alice"

show_user() {
    echo "User: $username"
}

show_user    # Output: User: Alice

# Function can modify global variable
change_user() {
    username="Bob"
}

change_user
echo "User is now: $username"    # Output: User is now: Bob
```

### Local Variables

Use `local` keyword to create function-scoped variables:

```bash
#!/bin/bash

# Global variable
count=10

test_scope() {
    local count=20    # Local to function
    echo "Inside function: $count"
}

test_scope               # Output: Inside function: 20
echo "Outside: $count"   # Output: Outside: 10
```

### Best Practices for Scope

```bash
# ✅ GOOD: Use local for function variables
calculate() {
    local num1=$1
    local num2=$2
    local result=$(( num1 + num2 ))
    echo $result
}

# ❌ BAD: Using globals everywhere
calculate_bad() {
    num1=$1    # Modifies global scope
    num2=$2
    result=$(( num1 + num2 ))
    echo $result
}

# ✅ GOOD: Explicit global modification
update_counter() {
    global_counter=$(( global_counter + 1 ))
}

# ✅ GOOD: Return values instead of modifying globals
get_timestamp() {
    local timestamp=$(date +%s)
    echo "$timestamp"
}

current_time=$(get_timestamp)
```

### Scope Comparison

```bash
#!/bin/bash

# Demonstrating scope
name="Global"

test_scope() {
    local name="Local"
    echo "Inside function: $name"
    
    # This creates a new local variable
    local temp="Temporary"
    echo "Temp inside: $temp"
}

test_scope
echo "Outside function: $name"
echo "Temp outside: $temp"    # Empty - not accessible

# Output:
# Inside function: Local
# Temp inside: Temporary
# Outside function: Global
# Temp outside:
```

---

## Recursive Functions

Functions can call themselves recursively:

### Factorial Example

```bash
factorial() {
    local n=$1
    
    # Base case
    if [ $n -le 1 ]; then
        echo 1
        return
    fi
    
    # Recursive case
    local prev=$(factorial $(( n - 1 )))
    echo $(( n * prev ))
}

result=$(factorial 5)
echo "5! = $result"    # Output: 5! = 120
```

### Fibonacci Sequence

```bash
fibonacci() {
    local n=$1
    
    if [ $n -le 1 ]; then
        echo $n
        return
    fi
    
    local a=$(fibonacci $(( n - 1 )))
    local b=$(fibonacci $(( n - 2 )))
    echo $(( a + b ))
}

# First 10 Fibonacci numbers
for i in {0..9}; do
    echo -n "$(fibonacci $i) "
done
echo
# Output: 0 1 1 2 3 5 8 13 21 34
```

### Directory Tree Recursion

```bash
list_tree() {
    local dir="$1"
    local indent="${2:-}"
    
    for item in "$dir"/*; do
        if [ -d "$item" ]; then
            echo "${indent}📁 $(basename "$item")"
            list_tree "$item" "${indent}  "
        else
            echo "${indent}📄 $(basename "$item")"
        fi
    done
}

list_tree "/path/to/directory"
```

### Recursion Caution

```bash
# ⚠️ Be careful with recursion depth
countdown() {
    local n=$1
    
    if [ $n -le 0 ]; then
        echo "Done!"
        return
    fi
    
    echo $n
    countdown $(( n - 1 ))
}

# This works for small numbers
countdown 10

# This might cause stack overflow
# countdown 10000    # Don't do this!
```

---

## Advanced Function Techniques

### Named Parameters

```bash
# Simulate named parameters
create_user() {
    local username=""
    local email=""
    local role="user"
    
    while [ $# -gt 0 ]; do
        case $1 in
            --username) username="$2"; shift 2;;
            --email) email="$2"; shift 2;;
            --role) role="$2"; shift 2;;
            *) shift;;
        esac
    done
    
    echo "Creating user: $username ($email) with role: $role"
}

create_user --username alice --email alice@example.com --role admin
```

### Function Arrays

```bash
# Array of function names
operations=("add" "subtract" "multiply" "divide")

add() { echo $(( $1 + $2 )); }
subtract() { echo $(( $1 - $2 )); }
multiply() { echo $(( $1 * $2 )); }
divide() { echo $(( $1 / $2 )); }

# Call functions dynamically
for op in "${operations[@]}"; do
    result=$($op 10 5)
    echo "$op 10 5 = $result"
done
```

### Callback Functions

```bash
# Function that accepts another function as callback
process_files() {
    local callback=$1
    shift
    
    for file in "$@"; do
        $callback "$file"
    done
}

# Callback functions
print_file() {
    echo "Processing: $1"
}

check_size() {
    local size=$(stat -f%z "$1" 2>/dev/null || stat -c%s "$1" 2>/dev/null)
    echo "$1: $size bytes"
}

# Use callbacks
process_files print_file *.txt
process_files check_size *.log
```

---

## Best Practices

### 1. Always Use Local Variables

```bash
# ✅ GOOD
calculate() {
    local num1=$1
    local num2=$2
    local result=$(( num1 + num2 ))
    echo $result
}

# ❌ BAD
calculate_bad() {
    result=$(( $1 + $2 ))    # Pollutes global scope
    echo $result
}
```

### 2. Validate Input

```bash
# ✅ GOOD: Check arguments
safe_division() {
    if [ $# -ne 2 ]; then
        echo "Error: Expected 2 arguments" >&2
        return 1
    fi
    
    if ! [[ "$1" =~ ^[0-9]+$ ]] || ! [[ "$2" =~ ^[0-9]+$ ]]; then
        echo "Error: Arguments must be numbers" >&2
        return 1
    fi
    
    if [ "$2" -eq 0 ]; then
        echo "Error: Division by zero" >&2
        return 1
    fi
    
    echo $(( $1 / $2 ))
}
```

### 3. Use Meaningful Names

```bash
# ✅ GOOD: Descriptive names
check_disk_space() { ... }
backup_database() { ... }
send_email_alert() { ... }

# ❌ BAD: Vague names
func1() { ... }
do_stuff() { ... }
x() { ... }
```

### 4. Document Functions

```bash
# Function documentation
##
# Backs up a directory to specified location
# Arguments:
#   $1 - Source directory
#   $2 - Destination directory
# Returns:
#   0 on success, 1 on failure
##
backup_directory() {
    local source="$1"
    local dest="$2"
    
    if [ ! -d "$source" ]; then
        echo "Error: Source directory not found" >&2
        return 1
    fi
    
    tar -czf "$dest/backup-$(date +%Y%m%d).tar.gz" "$source"
    return $?
}
```

### 5. Handle Errors Gracefully

```bash
# Error handling pattern
safe_operation() {
    local file="$1"
    
    # Check preconditions
    if [ -z "$file" ]; then
        echo "Error: No file specified" >&2
        return 1
    fi
    
    if [ ! -f "$file" ]; then
        echo "Error: File not found: $file" >&2
        return 1
    fi
    
    # Perform operation
    if ! cp "$file" "${file}.bak"; then
        echo "Error: Failed to create backup" >&2
        return 1
    fi
    
    echo "Success: Backed up $file"
    return 0
}
```

### 6. Keep Functions Focused

```bash
# ✅ GOOD: Single responsibility
is_valid_email() {
    [[ $1 =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]
}

send_email() {
    local to="$1"
    local subject="$2"
    local body="$3"
    
    echo "$body" | mail -s "$subject" "$to"
}

notify_user() {
    local email="$1"
    
    if is_valid_email "$email"; then
        send_email "$email" "Notification" "Task completed"
    fi
}

# ❌ BAD: Doing too much
process_everything() {
    # Validates, sends email, logs, updates database...
    # Too many responsibilities!
}
```

---

## Practical Examples

### Example 1: User Management Function

```bash
#!/bin/bash

create_system_user() {
    local username="$1"
    local home_dir="/home/$username"
    
    # Validate input
    if [ -z "$username" ]; then
        echo "Error: Username required" >&2
        return 1
    fi
    
    # Check if user exists
    if id "$username" &>/dev/null; then
        echo "Error: User $username already exists" >&2
        return 1
    fi
    
    # Create user
    if sudo useradd -m -d "$home_dir" -s /bin/bash "$username"; then
        echo "✓ User $username created successfully"
        echo "✓ Home directory: $home_dir"
        return 0
    else
        echo "Error: Failed to create user" >&2
        return 1
    fi
}

# Usage
create_system_user "newuser"
```

### Example 2: Log Rotation Function

```bash
#!/bin/bash

rotate_log() {
    local log_file="$1"
    local keep_days="${2:-7}"
    local timestamp=$(date +%Y%m%d_%H%M%S)
    
    if [ ! -f "$log_file" ]; then
        echo "Error: Log file not found: $log_file" >&2
        return 1
    fi
    
    # Create backup
    local backup="${log_file}.${timestamp}"
    cp "$log_file" "$backup"
    
    # Compress backup
    gzip "$backup"
    
    # Clear original log
    > "$log_file"
    
    # Delete old backups
    find "$(dirname "$log_file")" -name "$(basename "$log_file").*.gz" \
        -mtime +$keep_days -delete
    
    echo "✓ Rotated $log_file"
    return 0
}

# Usage
rotate_log "/var/log/application.log" 30
```

### Example 3: Service Health Check

```bash
#!/bin/bash

check_service() {
    local service_name="$1"
    
    if systemctl is-active --quiet "$service_name"; then
        echo "✓ $service_name is running"
        return 0
    else
        echo "✗ $service_name is not running"
        return 1
    fi
}

restart_service() {
    local service_name="$1"
    
    echo "Restarting $service_name..."
    if sudo systemctl restart "$service_name"; then
        sleep 2
        if check_service "$service_name"; then
            echo "✓ $service_name restarted successfully"
            return 0
        fi
    fi
    
    echo "✗ Failed to restart $service_name" >&2
    return 1
}

# Monitor and restart if needed
monitor_service() {
    local service="$1"
    
    if ! check_service "$service"; then
        echo "Service down, attempting restart..."
        restart_service "$service"
    fi
}

# Usage
monitor_service "nginx"
```

---

## Summary

In this chapter, you learned:

- ✅ **Function Basics**: How to declare and call functions using both syntaxes
- ✅ **Arguments**: Working with positional parameters ($1, $2, $@, $#)
- ✅ **Return Values**: Using return codes and echo for output
- ✅ **Scope**: Understanding local vs global variables
- ✅ **Recursion**: Creating recursive functions safely
- ✅ **Best Practices**: Writing clean, maintainable, and documented functions

### Key Takeaways

| Concept | Best Practice |
|---------|---------------|
| Declaration | Use `name() { ... }` syntax |
| Variables | Always use `local` for function variables |
| Arguments | Validate input before processing |
| Return | Use return codes for status, echo for data |
| Naming | Use descriptive, verb-based names |
| Documentation | Comment complex functions |

### Next Steps

In [Chapter 13: Modular Scripts](./13-modular-scripts.md), you'll learn how to organize functions across multiple files, create reusable libraries, and structure large Bash projects.

---

## Practice Exercises

1. **Exercise 1**: Create a function that calculates BMI (Body Mass Index) given weight (kg) and height (m)
2. **Exercise 2**: Write a function that validates IP addresses
3. **Exercise 3**: Create a recursive function to calculate the sum of digits in a number
4. **Exercise 4**: Build a menu system using functions for each menu option
5. **Exercise 5**: Write a function library for string operations (uppercase, lowercase, reverse)

**Solutions**: Try solving these yourself first, then check the answers in the appendix!
