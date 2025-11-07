# Chapter 9: Conditional Statements

## Table of Contents
1. [Introduction to Conditionals](#introduction-to-conditionals)
2. [if Statement](#if-statement)
3. [if-else Statement](#if-else-statement)
4. [if-elif-else Statement](#if-elif-else-statement)
5. [Test Conditions](#test-conditions)
6. [[ ]] vs [ ] Comparison](#-vs---comparison)
7. [Logical Operators](#logical-operators)
8. [Practical Examples](#practical-examples)
9. [Summary](#summary)

---

## Introduction to Conditionals

Conditional statements allow scripts to make decisions based on conditions.

```bash
# Basic structure
if [ condition ]; then
    # commands
fi

# With else
if [ condition ]; then
    # commands if true
else
    # commands if false
fi

# Multiple conditions
if [ condition1 ]; then
    # commands
elif [ condition2 ]; then
    # commands
else
    # commands
fi
```

---

## if Statement

### Basic Syntax

```bash
# Simple if
if [ -f file.txt ]; then
    echo "File exists"
fi

# Multi-line condition
if [ $age -ge 18 ]; then
    echo "Adult"
fi

# Compact form (one line)
[ -f file.txt ] && echo "File exists"

# Negation
if [ ! -f file.txt ]; then
    echo "File does not exist"
fi
```

### Common Patterns

```bash
# Check if variable is set
if [ -n "$VAR" ]; then
    echo "Variable is set"
fi

# Check if variable is empty
if [ -z "$VAR" ]; then
    echo "Variable is empty"
fi

# Check command success
if command; then
    echo "Command succeeded"
fi

# Check file exists
if [ -e "file.txt" ]; then
    echo "File exists"
fi
```

---

## if-else Statement

```bash
# Basic if-else
if [ $score -ge 60 ]; then
    echo "Pass"
else
    echo "Fail"
fi

# With command
if grep -q "error" logfile.txt; then
    echo "Errors found"
else
    echo "No errors"
fi

# File check
if [ -f config.conf ]; then
    source config.conf
else
    echo "Config file not found"
    exit 1
fi
```

---

## if-elif-else Statement

```bash
# Grade calculator
if [ $score -ge 90 ]; then
    grade="A"
elif [ $score -ge 80 ]; then
    grade="B"
elif [ $score -ge 70 ]; then
    grade="C"
elif [ $score -ge 60 ]; then
    grade="D"
else
    grade="F"
fi

echo "Grade: $grade"

# System check
if [ "$OS" = "Linux" ]; then
    package_manager="apt"
elif [ "$OS" = "macOS" ]; then
    package_manager="brew"
elif [ "$OS" = "Windows" ]; then
    package_manager="choco"
else
    echo "Unknown OS"
    exit 1
fi
```

---

## Test Conditions

### File Tests

```bash
# File exists
[ -e file.txt ]

# Regular file
[ -f file.txt ]

# Directory
[ -d /path/to/dir ]

# Symbolic link
[ -L symlink ]

# File readable
[ -r file.txt ]

# File writable
[ -w file.txt ]

# File executable
[ -x script.sh ]

# File not empty
[ -s file.txt ]

# Files are same
[ file1 -ef file2 ]

# File1 newer than file2
[ file1 -nt file2 ]

# File1 older than file2
[ file1 -ot file2 ]
```

### String Tests

```bash
# String is empty
[ -z "$str" ]

# String is not empty
[ -n "$str" ]

# Strings equal
[ "$str1" = "$str2" ]
[ "$str1" == "$str2" ]  # Same

# Strings not equal
[ "$str1" != "$str2" ]

# String less than (alphabetical)
[ "$str1" < "$str2" ]

# String greater than
[ "$str1" > "$str2" ]

# Pattern matching (requires [[]])
[[ "$str" == pattern* ]]
```

### Numeric Tests

```bash
# Equal
[ $a -eq $b ]

# Not equal
[ $a -ne $b ]

# Greater than
[ $a -gt $b ]

# Greater than or equal
[ $a -ge $b ]

# Less than
[ $a -lt $b ]

# Less than or equal
[ $a -le $b ]

# Example
if [ $age -ge 18 ] && [ $age -lt 65 ]; then
    echo "Working age"
fi
```

### Complete Test Reference

| Test | Description | Example |
|------|-------------|---------|
| `-e` | File exists | `[ -e file ]` |
| `-f` | Regular file | `[ -f file ]` |
| `-d` | Directory | `[ -d dir ]` |
| `-L` | Symbolic link | `[ -L link ]` |
| `-r` | Readable | `[ -r file ]` |
| `-w` | Writable | `[ -w file ]` |
| `-x` | Executable | `[ -x file ]` |
| `-s` | Not empty | `[ -s file ]` |
| `-z` | String empty | `[ -z "$str" ]` |
| `-n` | String not empty | `[ -n "$str" ]` |
| `=` | String equal | `[ "$a" = "$b" ]` |
| `!=` | String not equal | `[ "$a" != "$b" ]` |
| `-eq` | Number equal | `[ $a -eq $b ]` |
| `-ne` | Number not equal | `[ $a -ne $b ]` |
| `-gt` | Greater than | `[ $a -gt $b ]` |
| `-ge` | Greater or equal | `[ $a -ge $b ]` |
| `-lt` | Less than | `[ $a -lt $b ]` |
| `-le` | Less or equal | `[ $a -le $b ]` |

---

## [[ ]] vs [ ] Comparison

### Single Bracket [ ] (POSIX)

```bash
# Traditional test command
if [ -f file.txt ]; then
    echo "File exists"
fi

# Limitations:
# - Must quote variables
if [ "$var" = "value" ]; then
    echo "Match"
fi

# - Word splitting issues
var="two words"
if [ $var = "two words" ]; then  # ERROR!
    echo "Won't work"
fi

# - No pattern matching
if [ "$str" = pattern* ]; then  # Literal comparison!
    echo "Checks for literal asterisk"
fi
```

### Double Bracket [[ ]] (Bash Extended)

```bash
# Modern Bash test
if [[ -f file.txt ]]; then
    echo "File exists"
fi

# Advantages:

# 1. No need to quote variables
if [[ $var = value ]]; then
    echo "Works without quotes"
fi

# 2. Pattern matching
if [[ $str == pattern* ]]; then
    echo "Matches pattern"
fi

# 3. Regex matching
if [[ $email =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
    echo "Valid email"
fi

# 4. No word splitting
var="two words"
if [[ $var == "two words" ]]; then  # WORKS!
    echo "Matches"
fi

# 5. Logical operators work naturally
if [[ $a -gt 5 && $b -lt 10 ]]; then
    echo "Both conditions true"
fi
```

### When to Use Which

```bash
# Use [ ] when:
# - Writing portable POSIX scripts
# - Need to work on old systems
if [ "$1" = "start" ]; then
    start_service
fi

# Use [[ ]] when:
# - Writing Bash-specific scripts
# - Need pattern matching
# - Want safer variable handling
if [[ $filename == *.txt ]]; then
    process_text_file
fi
```

---

## Logical Operators

### AND Operator

```bash
# Using -a ([ ] only)
if [ $age -ge 18 -a $age -le 65 ]; then
    echo "Working age"
fi

# Using && (preferred with [[]])
if [[ $age -ge 18 && $age -le 65 ]]; then
    echo "Working age"
fi

# Separate if statements
if [ $age -ge 18 ] && [ $age -le 65 ]; then
    echo "Working age"
fi

# Command chaining
if [ -f file.txt ] && [ -r file.txt ]; then
    cat file.txt
fi
```

### OR Operator

```bash
# Using -o ([ ] only)
if [ "$answer" = "y" -o "$answer" = "yes" ]; then
    echo "Confirmed"
fi

# Using || (preferred with [[]])
if [[ $answer == "y" || $answer == "yes" ]]; then
    echo "Confirmed"
fi

# Separate conditions
if [ "$answer" = "y" ] || [ "$answer" = "yes" ]; then
    echo "Confirmed"
fi

# Pattern matching (only [[]])
if [[ $answer == [yY]* ]]; then
    echo "Confirmed"
fi
```

### NOT Operator

```bash
# Using !
if [ ! -f file.txt ]; then
    echo "File does not exist"
fi

if [[ ! $str == pattern* ]]; then
    echo "Doesn't match pattern"
fi

# Alternative for file tests
if [ -e file.txt ]; then
    :  # File exists
else
    echo "File doesn't exist"
fi
```

### Complex Conditions

```bash
# Grouping with && and ||
if [[ ($a -gt 5 && $b -lt 10) || $c -eq 0 ]]; then
    echo "Complex condition met"
fi

# Multiple file checks
if [[ -f input.txt && -r input.txt && -s input.txt ]]; then
    echo "File exists, is readable, and not empty"
fi

# Mixed operators
if [[ $user == "admin" || ($permissions == "rw" && $verified == true) ]]; then
    allow_access
fi
```

---

## Practical Examples

### File Validation

```bash
#!/bin/bash

validate_file() {
    local file="$1"
    
    if [[ ! -e $file ]]; then
        echo "Error: File does not exist: $file"
        return 1
    fi
    
    if [[ ! -f $file ]]; then
        echo "Error: Not a regular file: $file"
        return 1
    fi
    
    if [[ ! -r $file ]]; then
        echo "Error: File not readable: $file"
        return 1
    fi
    
    if [[ ! -s $file ]]; then
        echo "Warning: File is empty: $file"
    fi
    
    return 0
}

if validate_file "data.txt"; then
    echo "File validation passed"
    process_file "data.txt"
fi
```

### User Input Validation

```bash
#!/bin/bash

# Validate age
read -p "Enter age: " age

if [[ ! $age =~ ^[0-9]+$ ]]; then
    echo "Error: Age must be a number"
    exit 1
fi

if [[ $age -lt 0 || $age -gt 150 ]]; then
    echo "Error: Age out of range"
    exit 1
fi

# Validate email
read -p "Enter email: " email

if [[ ! $email =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
    echo "Error: Invalid email format"
    exit 1
fi

echo "Validation passed"
```

### System Checks

```bash
#!/bin/bash

# Check if running as root
if [[ $EUID -ne 0 ]]; then
    echo "This script must be run as root"
    exit 1
fi

# Check OS
if [[ -f /etc/debian_version ]]; then
    OS="Debian"
elif [[ -f /etc/redhat-release ]]; then
    OS="RedHat"
elif [[ "$OSTYPE" == "darwin"* ]]; then
    OS="macOS"
else
    echo "Unsupported OS"
    exit 1
fi

echo "Running on: $OS"

# Check disk space
available=$(df / | awk 'NR==2 {print $4}')
if [[ $available -lt 1000000 ]]; then
    echo "Warning: Low disk space"
fi

# Check if command exists
if command -v docker &> /dev/null; then
    echo "Docker is installed"
else
    echo "Docker not found"
fi
```

### Configuration Based Execution

```bash
#!/bin/bash

config_file="config.conf"

if [[ -f $config_file ]]; then
    source "$config_file"
else
    echo "Config not found, using defaults"
    DEBUG=false
    VERBOSE=false
    LOG_LEVEL="INFO"
fi

if [[ $DEBUG == true ]]; then
    set -x  # Enable debug mode
fi

if [[ $VERBOSE == true ]]; then
    echo "Verbose mode enabled"
fi

case "$LOG_LEVEL" in
    DEBUG|INFO|WARN|ERROR)
        echo "Log level: $LOG_LEVEL"
        ;;
    *)
        echo "Invalid log level, using INFO"
        LOG_LEVEL="INFO"
        ;;
esac
```

---

## Summary

### Basic Syntax

```bash
# if
if [ condition ]; then
    commands
fi

# if-else
if [ condition ]; then
    commands
else
    commands
fi

# if-elif-else
if [ condition1 ]; then
    commands
elif [ condition2 ]; then
    commands
else
    commands
fi
```

### Key Differences

| Feature | [ ] | [[ ]] |
|---------|-----|-------|
| POSIX | ✅ Yes | ❌ No (Bash only) |
| Quote variables | ✅ Required | ⚠️ Optional |
| Pattern matching | ❌ No | ✅ Yes |
| Regex | ❌ No | ✅ Yes |
| && and \|\| | ❌ Use -a/-o | ✅ Yes |
| Recommended | Old scripts | New Bash scripts |

### Common Patterns

```bash
# File exists
[[ -f file.txt ]] && echo "Found"

# Variable not empty
[[ -n $var ]] && echo "Set"

# Numeric comparison
[[ $a -gt $b ]] && echo "Greater"

# String pattern
[[ $file == *.txt ]] && echo "Text file"

# Multiple conditions
[[ $a -gt 5 && $b -lt 10 ]] && echo "In range"

# Command success
if command; then
    echo "Success"
fi
```

### Best Practices

1. Use `[[ ]]` for Bash scripts
2. Always quote variables in `[ ]`
3. Use meaningful variable names
4. Check for errors early
5. Provide clear error messages

---

**Next Chapter:** [Loops in Bash →](10-loops-in-bash.md)

---

*Practice Exercise: Create a script that validates user input, checks file permissions, and uses complex conditional logic to determine script behavior.*
