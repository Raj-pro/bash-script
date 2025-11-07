# Chapter 5: Working with Variables

## Table of Contents
1. [Introduction to Variables](#introduction-to-variables)
2. [Declaring and Using Variables](#declaring-and-using-variables)
3. [Environment vs User Variables](#environment-vs-user-variables)
4. [Command Substitution](#command-substitution)
5. [Variable Expansion](#variable-expansion)
6. [Special Variables](#special-variables)
7. [Best Practices](#best-practices)
8. [Summary](#summary)

---

## Introduction to Variables

Variables in Bash are used to store data that can be referenced and manipulated throughout your scripts.

**Key Characteristics:**
- No data type declaration needed (everything is a string by default)
- Case-sensitive (`VAR` and `var` are different)
- No spaces around `=` sign
- Use `$` to reference variable value

---

## Declaring and Using Variables

### Basic Variable Assignment

```bash
# Simple assignment
name="John"
age=25
path="/home/user"

# Using variables
echo $name          # Output: John
echo "Name: $name"  # Output: Name: John
echo "Age: $age"    # Output: Age: 25

# No spaces around =
# WRONG:
name = "John"      # Error: command not found
name= "John"       # Sets name to empty, runs "John" as command
name ="John"       # Error

# CORRECT:
name="John"
```

### Variable Naming Rules

```bash
# Valid variable names
my_var="value"
MyVar="value"
_var="value"
VAR123="value"

# Invalid variable names
# 123var="value"     # Cannot start with number
# my-var="value"     # Cannot contain hyphens
# my.var="value"     # Cannot contain dots
# my var="value"     # Cannot contain spaces
```

### Reading Variables

```bash
# Read from user input
read name
echo "Hello, $name"

# Read with prompt
read -p "Enter your name: " name

# Read password (silent input)
read -sp "Enter password: " password
echo

# Read multiple variables
read -p "Enter first and last name: " first last

# Read into array
read -a colors
echo "First color: ${colors[0]}"

# Read with timeout
read -t 5 -p "Quick! Enter something (5 sec): " response

# Read single character
read -n 1 -p "Press any key to continue..."
```

### Readonly Variables

```bash
# Create readonly variable
readonly PI=3.14159
declare -r CONST="Cannot change"

# Attempting to change causes error
PI=3.14  # Error: PI: readonly variable

# List all readonly variables
readonly
```

### Unsetting Variables

```bash
# Create variable
myvar="value"

# Remove variable
unset myvar

# Variable no longer exists
echo $myvar  # Empty output

# Cannot unset readonly variables
readonly CONST="value"
unset CONST  # Error: CONST: readonly variable
```

---

## Environment vs User Variables

### User (Local) Variables

**Scope:** Current shell only

```bash
# Set local variable
LOCAL_VAR="I am local"

# Start new shell
bash

# Variable not available
echo $LOCAL_VAR  # Empty

exit  # Return to parent shell
echo $LOCAL_VAR  # Output: I am local
```

### Environment Variables

**Scope:** Current shell and all child processes

```bash
# Set environment variable (method 1)
export GLOBAL_VAR="I am global"

# Set environment variable (method 2)
ANOTHER_VAR="value"
export ANOTHER_VAR

# Set and export in one line (method 3)
export YET_ANOTHER="value"

# Start new shell
bash

# Variable IS available
echo $GLOBAL_VAR  # Output: I am global

# List all environment variables
env
printenv
export

# Get specific variable
printenv HOME
echo $HOME
```

### Common Environment Variables

```bash
# System environment variables
echo $HOME          # User's home directory: /home/user
echo $USER          # Current username: user
echo $SHELL         # Default shell: /bin/bash
echo $PATH          # Executable search path
echo $PWD           # Current directory
echo $OLDPWD        # Previous directory
echo $HOSTNAME      # Computer name
echo $LANG          # System language
echo $EDITOR        # Default editor
echo $TERM          # Terminal type

# Display all
printenv
```

### Modifying PATH

```bash
# View current PATH
echo $PATH
# Output: /usr/local/bin:/usr/bin:/bin

# Add directory to PATH (temporary)
export PATH="$PATH:/home/user/bin"

# Add to beginning of PATH
export PATH="/home/user/bin:$PATH"

# Make permanent (add to ~/.bashrc)
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# Remove duplicates from PATH
export PATH=$(echo "$PATH" | awk -v RS=':' -v ORS=":" '!a[$1]++{if (NR > 1) printf ":"; printf "%s", $1}')
```

### Setting Default Values

```bash
# Use default if variable is unset
echo ${VAR:-"default value"}

# Set variable to default if unset
: ${VAR:="default value"}

# Display error if variable is unset
echo ${VAR:?"Error: VAR is not set"}

# Use alternate value if variable IS set
echo ${VAR:+"alternate value"}

# Examples
name=""
echo ${name:-"Unknown"}     # Output: Unknown
echo ${name:="John"}        # Sets name to "John"
echo $name                  # Output: John

# Empty vs Unset
empty=""
echo ${empty:-"default"}    # Output: default (empty)
echo ${unset:-"default"}    # Output: default (unset)
```

---

## Command Substitution

### Backticks vs $() Syntax

```bash
# Old syntax (backticks) - NOT recommended
current_date=`date`
files=`ls`

# Modern syntax - RECOMMENDED
current_date=$(date)
files=$(ls)

# Why $() is better:
# 1. Easier to read
# 2. Can be nested
# 3. Easier to escape
# 4. More consistent

# Nested command substitution
# HARD to read with backticks:
outer=`echo \`echo hello\``

# EASY to read with $():
outer=$(echo $(echo hello))
```

### Practical Examples

```bash
# Store command output
current_user=$(whoami)
current_dir=$(pwd)
file_count=$(ls | wc -l)

# Use in strings
echo "User $current_user is in $current_dir"
echo "There are $file_count files here"

# Store multi-line output
processes=$(ps aux)

# Use in conditions
if [ $(whoami) = "root" ]; then
    echo "Running as root"
fi

# Arithmetic
total=$(expr 5 + 3)
total=$((5 + 3))  # Better way

# Get current date/time
timestamp=$(date +%Y%m%d_%H%M%S)
filename="backup_${timestamp}.tar.gz"

# Count files
txt_files=$(find . -name "*.txt" | wc -l)
echo "Found $txt_files text files"

# Get system information
kernel=$(uname -r)
memory=$(free -h | awk '/^Mem:/ {print $2}')
```

---

## Variable Expansion

### Basic Expansion

```bash
name="John"

# Basic expansion
echo $name        # Output: John
echo "$name"      # Output: John
echo '$name'      # Output: $name (literal)

# Braces for clarity
echo "User: ${name}"
echo "File: ${name}.txt"

# Without braces (can be ambiguous)
echo "File: $name.txt"  # Works if variable is "name"
echo "File: $nametest"  # Looks for variable "nametest"
```

### String Length

```bash
text="Hello World"

# Get length
echo ${#text}     # Output: 11

# Get array length
arr=(one two three)
echo ${#arr[@]}   # Output: 3
```

### Substring Extraction

```bash
text="Hello World"

# Extract substring: ${var:position:length}
echo ${text:0:5}   # Output: Hello
echo ${text:6}     # Output: World
echo ${text:6:3}   # Output: Wor
echo ${text: -5}   # Output: World (last 5 chars, note space)

# Practical example
filename="document.txt"
extension="${filename: -3}"  # Output: txt
basename="${filename:0: -4}" # Output: document
```

### Pattern Removal

```bash
filename="path/to/file.tar.gz"

# Remove from beginning (shortest match)
echo ${filename#*/}      # Output: to/file.tar.gz

# Remove from beginning (longest match)
echo ${filename##*/}     # Output: file.tar.gz

# Remove from end (shortest match)
echo ${filename%.*}      # Output: path/to/file.tar

# Remove from end (longest match)
echo ${filename%%.*}     # Output: path/to/file

# Practical examples
path="/home/user/documents/file.txt"
basename=${path##*/}     # file.txt
dirname=${path%/*}       # /home/user/documents

url="https://example.com/page.html"
protocol=${url%%://*}    # https
domain=${url#*://}       # example.com/page.html
domain=${domain%%/*}     # example.com
```

### Pattern Replacement

```bash
text="Hello World World"

# Replace first occurrence
echo ${text/World/Universe}  # Hello Universe World

# Replace all occurrences
echo ${text//World/Universe} # Hello Universe Universe

# Replace at beginning
echo ${text/#Hello/Hi}       # Hi World World

# Replace at end
echo ${text/%World/Universe} # Hello World Universe

# Delete pattern
echo ${text//World/}         # Hello  

# Practical examples
path="/home/user/docs"
echo ${path//\//\\}          # \home\user\docs (Windows path)

csv="a,b,c,d"
echo ${csv//,/ }             # a b c d (spaces instead of commas)
```

### Case Conversion (Bash 4+)

```bash
text="Hello World"

# Convert to uppercase
echo ${text^^}               # HELLO WORLD
echo ${text^^[aeiou]}        # HEllO WOrld (vowels only)

# Convert to lowercase
echo ${text,,}               # hello world
echo ${text,,[HW]}           # hello world (H and W only)

# Toggle first character
echo ${text^}                # Hello World
echo ${text,}                # hello World
```

---

## Special Variables

### Positional Parameters

```bash
#!/bin/bash
# script.sh

echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "All arguments: $@"
echo "All arguments (quoted): $*"
echo "Number of arguments: $#"
echo "Process ID: $$"
echo "Last command exit status: $?"
echo "Last background PID: $!"

# Usage: ./script.sh arg1 arg2 arg3
```

### Special Variables Table

| Variable | Description | Example |
|----------|-------------|---------|
| `$0` | Script name | `./myscript.sh` |
| `$1-$9` | Positional arguments 1-9 | `$1` = first arg |
| `${10}` | Arguments 10+ (use braces) | `${10}` = 10th arg |
| `$#` | Number of arguments | `3` |
| `$@` | All arguments (separate) | `"$1" "$2" "$3"` |
| `$*` | All arguments (single string) | `"$1 $2 $3"` |
| `$$` | Current shell PID | `12345` |
| `$!` | Last background process PID | `12346` |
| `$?` | Exit status of last command | `0` (success) |
| `$-` | Current shell options | `himBH` |
| `$_` | Last argument of last command | `file.txt` |

### $@ vs $* Difference

```bash
#!/bin/bash

# Without quotes - same behavior
echo "Using \$@:"
for arg in $@; do
    echo "  - $arg"
done

echo "Using \$*:"
for arg in $*; do
    echo "  - $arg"
done

# With quotes - different behavior
echo "Using \"\$@\":"
for arg in "$@"; do
    echo "  - $arg"
done

echo "Using \"\$*\":"
for arg in "$*"; do
    echo "  - $arg"
done

# Run: ./script.sh "arg 1" "arg 2" "arg 3"
# "$@" preserves separate arguments
# "$*" combines into single argument
```

### Exit Status ($?)

```bash
# Check if command succeeded
ls /tmp
echo $?  # 0 = success

ls /nonexistent
echo $?  # Non-zero = error

# Use in conditions
if grep "pattern" file.txt; then
    echo "Found"
else
    echo "Not found (exit status: $?)"
fi

# Chain commands
command1 && echo "Success" || echo "Failed"

# Custom exit codes in scripts
#!/bin/bash
if [ ! -f "file.txt" ]; then
    echo "Error: file.txt not found"
    exit 1  # Exit with error code
fi
exit 0  # Exit successfully
```

---

## Best Practices

### 1. Always Quote Variables

```bash
# BAD - can cause word splitting
file="my file.txt"
cat $file  # Error: tries to cat "my" and "file.txt" separately

# GOOD
cat "$file"  # Works correctly

# BAD - pathname expansion
pattern="*.txt"
rm $pattern  # Expands before rm sees it

# GOOD (if you want literal)
rm "$pattern"
```

### 2. Use Meaningful Names

```bash
# BAD
x="value"
tmp="/var/log"
a=5

# GOOD
username="john"
log_directory="/var/log"
max_retries=5
```

### 3. Use ${} for Clarity

```bash
# Ambiguous
echo "File: $filename_backup"  # Looks for variable "filename_backup"

# Clear
echo "File: ${filename}_backup"  # Uses "filename" variable
```

### 4. Initialize Variables

```bash
# BAD - undefined variable
if [ "$count" -gt 0 ]; then
    echo "Count is positive"
fi

# GOOD
count=0
if [ "$count" -gt 0 ]; then
    echo "Count is positive"
fi

# Use defaults
count=${count:-0}
```

### 5. Use Local Variables in Functions

```bash
# BAD - global variable
my_function() {
    result="global change"
}

# GOOD - local variable
my_function() {
    local result="local only"
}
```

### 6. Uppercase for Environment Variables

```bash
# Convention
export PATH="/usr/local/bin:$PATH"
export USER_HOME="/home/user"

# Lowercase for local variables
file_count=10
temp_dir="/tmp/myapp"
```

---

## Summary

### Key Takeaways

1. **Variables** store data as strings by default
2. **No spaces** around `=` in assignments
3. **$** prefix to access variable values
4. **export** makes variables available to child processes
5. **Command substitution** with `$()` (preferred) or backticks
6. **Special variables** like `$?`, `$#`, `$@` provide useful information
7. **Always quote** variables to prevent word splitting

### Quick Reference

```bash
# Declaration
var="value"
readonly CONST="constant"
export GLOBAL="environment variable"

# Reading
read var
read -p "Prompt: " var
read -sp "Password: " pass

# Command substitution
result=$(command)

# Variable expansion
${var}              # Value
${#var}             # Length
${var:pos:len}      # Substring
${var#pattern}      # Remove from start
${var%pattern}      # Remove from end
${var/old/new}      # Replace
${var:-default}     # Default if unset
${var^^}            # Uppercase
${var,,}            # Lowercase

# Special variables
$0, $1, $2...       # Arguments
$#                  # Argument count
$@                  # All arguments
$?                  # Exit status
$$                  # Process ID
```

### Common Patterns

```bash
# Safe variable usage
filename="${1:-default.txt}"
count="${count:-0}"

# Build paths
log_file="${LOG_DIR}/${APP_NAME}_$(date +%Y%m%d).log"

# String manipulation
basename="${filepath##*/}"
extension="${filename##*.}"

# Check if variable is set
if [ -z "$VAR" ]; then
    echo "VAR is not set or empty"
fi
```

---

**Next Chapter:** [Quoting and Escaping →](06-quoting-and-escaping.md)

---

*Practice Exercise: Create a script that reads user input, uses command substitution to get system info, and demonstrates variable expansion techniques.*
