# Chapter 17: String Manipulation

## Introduction

String manipulation is essential for text processing and data transformation. This chapter covers substring extraction, pattern matching, and parameter expansion techniques.

---

## String Length

### Get String Length

```bash
#!/bin/bash

text="Hello World"

# Get length
length=${#text}
echo "Length: $length"  # Output: 11

# Works with variables
name="John"
echo "Name length: ${#name}"  # Output: 4

# Empty string
empty=""
echo "Empty length: ${#empty}"  # Output: 0
```

---

## Substring Extraction

### Basic Substring

```bash
#!/bin/bash

text="Hello World"

# Extract from position (0-indexed)
# ${variable:position:length}

# Get first 5 characters
echo "${text:0:5}"  # Hello

# Get from position 6 to end
echo "${text:6}"    # World

# Get last 5 characters (negative position)
echo "${text: -5}"  # World

# Get 3 characters from position 6
echo "${text:6:3}"  # Wor
```

### Practical Examples

```bash
#!/bin/bash

# Extract date components
date_string="2025-11-08"

year="${date_string:0:4}"
month="${date_string:5:2}"
day="${date_string:8:2}"

echo "Year: $year"    # 2025
echo "Month: $month"  # 11
echo "Day: $day"      # 08

# Extract file extension
filename="document.pdf"
extension="${filename: -3}"
echo "Extension: $extension"  # pdf

# Extract username from email
email="user@example.com"
username="${email:0:${email%%@*}}"
# Better way:
username="${email%%@*}"
echo "Username: $username"  # user
```

---

## Pattern Matching and Removal

### Remove from Beginning (# and ##)

```bash
#!/bin/bash

path="/home/user/documents/file.txt"

# Remove shortest match from beginning
echo "${path#*/}"        # home/user/documents/file.txt

# Remove longest match from beginning
echo "${path##*/}"       # file.txt (basename)

# Practical examples
url="https://www.example.com/page"
echo "${url#*://}"       # www.example.com/page
echo "${url##*/}"        # page

# Remove file extension
filename="document.tar.gz"
echo "${filename#*.}"    # tar.gz (removes first extension)
echo "${filename##*.}"   # gz (removes all extensions)
```

### Remove from End (% and %%)

```bash
#!/bin/bash

path="/home/user/documents/file.txt"

# Remove shortest match from end
echo "${path%/*}"        # /home/user/documents (dirname)

# Remove longest match from end
echo "${path%%/*}"       # (empty - removes everything)

# Practical examples
filename="document.tar.gz"
echo "${filename%.*}"    # document.tar (removes last extension)
echo "${filename%%.*}"   # document (removes all extensions)

# Remove trailing slash
path="/var/log/"
echo "${path%/}"         # /var/log

# Extract domain from URL
url="https://www.example.com/page"
domain="${url#*://}"
domain="${domain%%/*}"
echo "$domain"           # www.example.com
```

---

## String Replacement

### Simple Replacement

```bash
#!/bin/bash

text="Hello World"

# Replace first occurrence
echo "${text/World/Universe}"      # Hello Universe

# Replace all occurrences
text="foo bar foo"
echo "${text//foo/baz}"           # baz bar baz

# Replace at beginning
text="Hello World"
echo "${text/#Hello/Hi}"          # Hi World

# Replace at end
echo "${text/%World/Universe}"    # Hello Universe
```

### Practical Replacement Examples

```bash
#!/bin/bash

# Convert spaces to underscores
filename="My Document File.txt"
safe_filename="${filename// /_}"
echo "$safe_filename"  # My_Document_File.txt

# Remove all spaces
text="Hello   World   Test"
no_spaces="${text// /}"
echo "$no_spaces"      # HelloWorldTest

# Replace multiple characters
path="/home//user///documents"
clean_path="${path//\/\//\/}"
echo "$clean_path"     # /home/user/documents

# Convert to lowercase (bash 4+)
text="HELLO WORLD"
lowercase="${text,,}"
echo "$lowercase"      # hello world

# Convert to uppercase (bash 4+)
text="hello world"
uppercase="${text^^}"
echo "$uppercase"      # HELLO WORLD

# Capitalize first letter
text="hello"
capitalized="${text^}"
echo "$capitalized"    # Hello
```

---

## Pattern Matching

### Case Conversion

```bash
#!/bin/bash

text="Hello World"

# Lowercase (bash 4+)
echo "${text,,}"       # hello world
echo "${text,}"        # hello World (first char only)

# Uppercase (bash 4+)
echo "${text^^}"       # HELLO WORLD
echo "${text^}"        # Hello World (first char only)

# Toggle case
echo "${text~~}"       # hELLO wORLD
echo "${text~}"        # hello World (first char only)

# For older bash versions, use tr
echo "$text" | tr '[:upper:]' '[:lower:]'  # hello world
echo "$text" | tr '[:lower:]' '[:upper:]'  # HELLO WORLD
```

### Wildcard Matching

```bash
#!/bin/bash

filename="document.pdf"

# Check if string matches pattern
if [[ $filename == *.pdf ]]; then
    echo "PDF file"
fi

# Case-insensitive matching (bash 4+)
shopt -s nocasematch
if [[ $filename == *.PDF ]]; then
    echo "PDF file (case-insensitive)"
fi
shopt -u nocasematch

# Multiple patterns
if [[ $filename == *.@(pdf|doc|txt) ]]; then
    echo "Document file"
fi
```

---

## Default Values and Alternatives

### Parameter Expansion for Defaults

```bash
#!/bin/bash

# Use default if unset or null
name=""
echo "${name:-Anonymous}"      # Anonymous
echo "$name"                   # (still empty)

# Assign default if unset or null
echo "${name:=Anonymous}"      # Anonymous
echo "$name"                   # Anonymous (now set)

# Use alternative if set
name="John"
echo "${name:+Mr. $name}"      # Mr. John

# Error if unset or null
# echo "${required_var:?Error: Variable not set}"

# Practical examples
config_file="${1:-config.ini}"
log_level="${LOG_LEVEL:-INFO}"
port="${PORT:=8080}"
```

---

## String Trimming

### Remove Whitespace

```bash
#!/bin/bash

# Trim leading whitespace
text="   Hello World"
trimmed="${text#"${text%%[![:space:]]*}"}"
echo "[$trimmed]"  # [Hello World]

# Trim trailing whitespace
text="Hello World   "
trimmed="${text%"${text##*[![:space:]]}"}"
echo "[$trimmed]"  # [Hello World]

# Trim both (function)
trim() {
    local var="$*"
    # Remove leading whitespace
    var="${var#"${var%%[![:space:]]*}"}"
    # Remove trailing whitespace
    var="${var%"${var##*[![:space:]]}"}"
    echo "$var"
}

text="   Hello World   "
echo "[$(trim "$text")]"  # [Hello World]

# Using sed
echo "$text" | sed 's/^[[:space:]]*//;s/[[:space:]]*$//'
```

---

## String Validation

### Validate String Patterns

```bash
#!/bin/bash

# Check if string contains only numbers
is_numeric() {
    [[ $1 =~ ^[0-9]+$ ]]
}

# Check if string contains only letters
is_alpha() {
    [[ $1 =~ ^[a-zA-Z]+$ ]]
}

# Check if string is alphanumeric
is_alphanumeric() {
    [[ $1 =~ ^[a-zA-Z0-9]+$ ]]
}

# Check if valid email
is_email() {
    [[ $1 =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]
}

# Usage
if is_numeric "12345"; then
    echo "Valid number"
fi

if is_email "user@example.com"; then
    echo "Valid email"
fi
```

---

## Advanced String Operations

### Joining Strings

```bash
#!/bin/bash

# Simple concatenation
first="Hello"
last="World"
full="$first $last"
echo "$full"  # Hello World

# Join array elements
fruits=("apple" "banana" "cherry")

# Join with delimiter
IFS=","
joined="${fruits[*]}"
echo "$joined"  # apple,banana,cherry

# Join function
join_by() {
    local delimiter=$1
    shift
    local first=$1
    shift
    printf "%s" "$first" "${@/#/$delimiter}"
}

result=$(join_by ", " "${fruits[@]}")
echo "$result"  # apple, banana, cherry
```

### Splitting Strings

```bash
#!/bin/bash

# Split by delimiter
text="apple,banana,cherry"

# Read into array
IFS=',' read -ra fruits <<< "$text"
for fruit in "${fruits[@]}"; do
    echo "$fruit"
done

# Split into variables
IFS=',' read -r first second third <<< "$text"
echo "First: $first"   # apple
echo "Second: $second" # banana
echo "Third: $third"   # cherry

# Split by whitespace
text="one two three"
read -ra words <<< "$text"
echo "Count: ${#words[@]}"  # 3
```

---

## String Comparison

### Compare Strings

```bash
#!/bin/bash

str1="hello"
str2="world"

# Equality
if [ "$str1" = "$str2" ]; then
    echo "Equal"
else
    echo "Not equal"
fi

# Inequality
if [ "$str1" != "$str2" ]; then
    echo "Not equal"
fi

# Lexicographic comparison
if [[ "$str1" < "$str2" ]]; then
    echo "$str1 comes before $str2"
fi

# Check if empty
if [ -z "$str1" ]; then
    echo "String is empty"
fi

# Check if not empty
if [ -n "$str1" ]; then
    echo "String is not empty"
fi
```

---

## Practical Examples

### Parse CSV Data

```bash
#!/bin/bash

csv_line="John,Doe,30,Engineer"

# Parse CSV
IFS=',' read -r first last age job <<< "$csv_line"

echo "Name: $first $last"
echo "Age: $age"
echo "Job: $job"

# Process CSV file
while IFS=',' read -r first last age job; do
    echo "Processing: $first $last ($age) - $job"
done < data.csv
```

### URL Parsing

```bash
#!/bin/bash

parse_url() {
    local url=$1
    
    # Extract protocol
    protocol="${url%%://*}"
    
    # Remove protocol
    temp="${url#*://}"
    
    # Extract domain and path
    domain="${temp%%/*}"
    path="/${temp#*/}"
    
    # Extract port if present
    if [[ $domain == *:* ]]; then
        port="${domain##*:}"
        domain="${domain%%:*}"
    else
        port="80"
    fi
    
    echo "Protocol: $protocol"
    echo "Domain: $domain"
    echo "Port: $port"
    echo "Path: $path"
}

parse_url "https://example.com:8080/api/users"
```

### File Path Manipulation

```bash
#!/bin/bash

filepath="/home/user/documents/file.txt"

# Extract components
dirname="${filepath%/*}"         # /home/user/documents
basename="${filepath##*/}"       # file.txt
filename="${basename%.*}"        # file
extension="${basename##*.}"      # txt

echo "Directory: $dirname"
echo "Basename: $basename"
echo "Filename: $filename"
echo "Extension: $extension"

# Change extension
new_file="${filepath%.*}.pdf"
echo "New file: $new_file"  # /home/user/documents/file.pdf
```

### Generate Slug from Title

```bash
#!/bin/bash

create_slug() {
    local title=$1
    local slug
    
    # Convert to lowercase
    slug="${title,,}"
    
    # Replace spaces with hyphens
    slug="${slug// /-}"
    
    # Remove special characters
    slug=$(echo "$slug" | sed 's/[^a-z0-9-]//g')
    
    # Remove multiple hyphens
    slug=$(echo "$slug" | sed 's/--*/-/g')
    
    # Remove leading/trailing hyphens
    slug="${slug#-}"
    slug="${slug%-}"
    
    echo "$slug"
}

title="Hello World! This is a Test."
slug=$(create_slug "$title")
echo "$slug"  # hello-world-this-is-a-test
```

### Password Generator

```bash
#!/bin/bash

generate_password() {
    local length=${1:-16}
    
    # Using /dev/urandom
    password=$(tr -dc 'A-Za-z0-9!@#$%^&*' < /dev/urandom | head -c "$length")
    echo "$password"
}

# Generate 20-character password
password=$(generate_password 20)
echo "Password: $password"

# Validate password strength
validate_password() {
    local pass=$1
    
    # Check length
    if [ ${#pass} -lt 8 ]; then
        echo "Too short"
        return 1
    fi
    
    # Check for uppercase
    if ! [[ $pass =~ [A-Z] ]]; then
        echo "Missing uppercase"
        return 1
    fi
    
    # Check for lowercase
    if ! [[ $pass =~ [a-z] ]]; then
        echo "Missing lowercase"
        return 1
    fi
    
    # Check for digit
    if ! [[ $pass =~ [0-9] ]]; then
        echo "Missing digit"
        return 1
    fi
    
    echo "Strong password"
    return 0
}
```

---

## Performance Tips

1. **Use parameter expansion** over external commands (faster)
2. **Avoid subshells** when possible
3. **Use `${var//pattern/replacement}`** instead of `sed` for simple replacements
4. **Quote variables** to prevent word splitting
5. **Use `[[  ]]`** for string comparisons (more features)

```bash
# SLOW
result=$(echo "$var" | sed 's/old/new/g')

# FAST
result="${var//old/new}"

# SLOW
length=$(echo -n "$var" | wc -c)

# FAST
length=${#var}
```

---

## Summary

- Use `${#var}` for string length
- Use `${var:pos:len}` for substring extraction
- Use `#`, `##`, `%`, `%%` for pattern removal
- Use `//`, `/#`, `/%` for string replacement
- Use parameter expansion for default values
- Use regex with `[[ =~ ]]` for validation
- Prefer built-in operations over external commands

---

## Practice Exercises

1. Write a function to reverse a string
2. Create a slug generator for blog post URLs
3. Parse and validate email addresses
4. Extract all numbers from a string
5. Implement a simple template engine with variable substitution

---

**Next Chapter:** [18 - Date, Time, and Scheduling](18-date-time-scheduling.md)
