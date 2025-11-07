# Chapter 19: Regular Expressions in Bash

## Introduction

Regular expressions (regex) are powerful patterns used for matching and manipulating text. This chapter covers regex fundamentals and their use in Bash with tools like `grep`, `sed`, and `awk`.

---

## Regular Expression Basics

### What is a Regular Expression?

A pattern that describes a set of strings. Used for:
- Searching text
- Validating input
- Extracting data
- Text replacement

### Basic vs Extended Regex

- **Basic (BRE)**: Default in `grep`, `sed` - metacharacters need escaping
- **Extended (ERE)**: Used with `-E` flag - more metacharacters without escaping
- **Perl (PCRE)**: Most powerful, used with `-P` flag in `grep`

---

## Regex Metacharacters

### Literal Characters

```bash
# Match exact string
echo "hello world" | grep "hello"

# Case-insensitive matching
echo "Hello World" | grep -i "hello"
```

### Special Characters

```bash
.     # Any single character (except newline)
^     # Start of line
$     # End of line
*     # Zero or more of previous
+     # One or more of previous (ERE)
?     # Zero or one of previous (ERE)
[]    # Character class
[^]   # Negated character class
\     # Escape special character
|     # Alternation (OR) (ERE)
()    # Grouping (ERE)
{}    # Quantifier (ERE)
```

---

## Basic Pattern Matching

### Anchors

```bash
# ^ - Start of line
echo "hello world" | grep "^hello"  # Matches
echo "say hello" | grep "^hello"    # No match

# $ - End of line
echo "world" | grep "world$"        # Matches
echo "world map" | grep "world$"    # No match

# Both anchors - exact line match
grep "^exact line$" file.txt
```

### The Dot (.) - Any Character

```bash
# . matches any single character
echo "cat" | grep "c.t"     # Matches: cat
echo "cot" | grep "c.t"     # Matches: cot
echo "ct" | grep "c.t"      # No match (need one char)

# Escape . to match literal dot
echo "file.txt" | grep "file\.txt"
```

### Character Classes

```bash
# [abc] - Match any one of a, b, or c
echo "cat" | grep "[cb]at"    # Matches
echo "bat" | grep "[cb]at"    # Matches
echo "rat" | grep "[cb]at"    # No match

# [a-z] - Range
echo "hello" | grep "[a-z]"   # Matches lowercase
echo "HELLO" | grep "[A-Z]"   # Matches uppercase
echo "Test123" | grep "[0-9]" # Matches digits

# [^abc] - Negation (NOT a, b, or c)
echo "cat" | grep "[^r]at"    # Matches (not rat)
echo "rat" | grep "[^r]at"    # No match
```

### Predefined Character Classes

```bash
[:alnum:]   # Alphanumeric [a-zA-Z0-9]
[:alpha:]   # Alphabetic [a-zA-Z]
[:digit:]   # Digits [0-9]
[:lower:]   # Lowercase [a-z]
[:upper:]   # Uppercase [A-Z]
[:space:]   # Whitespace
[:punct:]   # Punctuation

# Usage
grep "[[:digit:]]" file.txt    # Find lines with digits
grep "^[[:upper:]]" file.txt   # Lines starting with uppercase
```

---

## Quantifiers

### Basic Quantifiers

```bash
# * - Zero or more
echo "ct" | grep "ca*t"       # Matches (zero a's)
echo "cat" | grep "ca*t"      # Matches (one a)
echo "caat" | grep "ca*t"     # Matches (two a's)

# \+ - One or more (BRE requires backslash)
echo "ct" | grep "ca\+t"      # No match
echo "cat" | grep "ca\+t"     # Matches
echo "caat" | grep "ca\+t"    # Matches

# \? - Zero or one (BRE requires backslash)
echo "color" | grep "colou\?r"    # Matches color
echo "colour" | grep "colou\?r"   # Matches colour
```

### Extended Quantifiers (ERE)

```bash
# Use -E flag for extended regex
grep -E "pattern" file.txt

# + - One or more (no backslash needed)
echo "cat" | grep -E "ca+t"

# ? - Zero or one
echo "color" | grep -E "colou?r"

# {n} - Exactly n times
echo "caaat" | grep -E "ca{3}t"   # Matches (exactly 3 a's)

# {n,} - n or more times
echo "caaaat" | grep -E "ca{2,}t" # Matches (2 or more a's)

# {n,m} - Between n and m times
echo "caat" | grep -E "ca{1,3}t"  # Matches (1-3 a's)
```

---

## Grouping and Alternation

### Grouping with Parentheses

```bash
# () - Group expressions (ERE)
echo "abcabc" | grep -E "(abc)+"       # Matches repeated abc
echo "testtest" | grep -E "(test){2}"  # Matches test twice

# Backreferences (in sed, not grep)
echo "hello hello" | sed -E 's/(hello) \1/\1/'  # Removes duplicate
```

### Alternation (OR)

```bash
# | - Match either pattern (ERE)
grep -E "cat|dog" file.txt        # Lines with cat OR dog
grep -E "^(root|admin):" /etc/passwd  # Lines starting with root or admin

# Combine with grouping
echo "I like cats" | grep -E "I like (cat|dog)s"  # Matches
```

---

## Regex in Bash Conditionals

### Using [[ ]] with =~

```bash
#!/bin/bash

# Pattern matching with =~
if [[ "test@example.com" =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
    echo "Valid email"
fi

# Extract matched groups
text="User: john123"
if [[ $text =~ User:\ ([a-z]+)([0-9]+) ]]; then
    echo "Username: ${BASH_REMATCH[1]}"  # john
    echo "Number: ${BASH_REMATCH[2]}"    # 123
fi
```

### Validation Examples

```bash
#!/bin/bash

# IP address validation
is_valid_ip() {
    local ip=$1
    local regex='^([0-9]{1,3}\.){3}[0-9]{1,3}$'
    [[ $ip =~ $regex ]]
}

# Phone number validation
is_valid_phone() {
    local phone=$1
    local regex='^\+?[0-9]{1,3}[-. ]?(\([0-9]{3}\)|[0-9]{3})[-. ]?[0-9]{3}[-. ]?[0-9]{4}$'
    [[ $phone =~ $regex ]]
}

# Usage
if is_valid_ip "192.168.1.1"; then
    echo "Valid IP"
fi
```

---

## Using grep with Regex

### Basic grep Usage

```bash
# Simple pattern
grep "pattern" file.txt

# Case-insensitive
grep -i "pattern" file.txt

# Invert match (lines NOT matching)
grep -v "pattern" file.txt

# Show line numbers
grep -n "pattern" file.txt

# Count matches
grep -c "pattern" file.txt

# Show only matching part
grep -o "pattern" file.txt
```

### Extended grep

```bash
# Use extended regex
grep -E "pattern1|pattern2" file.txt

# Recursive search
grep -r "pattern" /path/to/dir

# Search multiple files
grep "pattern" file1.txt file2.txt

# Show context
grep -A 2 "pattern" file.txt  # 2 lines after
grep -B 2 "pattern" file.txt  # 2 lines before
grep -C 2 "pattern" file.txt  # 2 lines before and after
```

### Practical grep Examples

```bash
# Find email addresses
grep -E "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" file.txt

# Find IP addresses
grep -E "([0-9]{1,3}\.){3}[0-9]{1,3}" file.txt

# Find URLs
grep -E "https?://[a-zA-Z0-9./?=_-]*" file.txt

# Find errors in logs
grep -E "(ERROR|FATAL|CRITICAL)" /var/log/app.log

# Find empty lines
grep "^$" file.txt

# Find lines with only whitespace
grep "^[[:space:]]*$" file.txt
```

---

## Using sed with Regex

### Basic Substitution

```bash
# s/pattern/replacement/
echo "hello world" | sed 's/world/universe/'

# Global replacement (all occurrences)
echo "cat cat cat" | sed 's/cat/dog/g'

# Case-insensitive
echo "Hello World" | sed 's/hello/hi/I'
```

### Advanced sed Patterns

```bash
# Delete lines matching pattern
sed '/pattern/d' file.txt

# Delete empty lines
sed '/^$/d' file.txt

# Print only matching lines
sed -n '/pattern/p' file.txt

# Replace with backreferences
echo "John Doe" | sed -E 's/([A-Z][a-z]+) ([A-Z][a-z]+)/\2, \1/'
# Output: Doe, John

# Multiple commands
sed -e 's/cat/dog/g' -e 's/bird/fish/g' file.txt
```

### Practical sed Examples

```bash
# Remove comments from config file
sed 's/#.*$//' config.txt

# Extract email domain
echo "user@example.com" | sed -E 's/.*@//'

# Convert DOS to Unix line endings
sed 's/\r$//' dosfile.txt > unixfile.txt

# Add line numbers
sed = file.txt | sed 'N;s/\n/\t/'

# Replace multiple spaces with single space
echo "hello    world" | sed 's/  */ /g'
```

---

## Using awk with Regex

### Basic awk Pattern Matching

```bash
# Print lines matching pattern
awk '/pattern/' file.txt

# Pattern with action
awk '/pattern/ {print $1}' file.txt

# Multiple patterns
awk '/pattern1/ || /pattern2/' file.txt
```

### Field Matching

```bash
# Match specific field
awk '$1 ~ /pattern/' file.txt     # First field matches
awk '$2 !~ /pattern/' file.txt    # Second field doesn't match

# Case-insensitive
awk 'tolower($0) ~ /pattern/' file.txt
```

### Practical awk Examples

```bash
# Extract email addresses
awk '/[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}/ {print $0}' file.txt

# Parse log file
awk '/ERROR/ {print $1, $2, $NF}' /var/log/app.log

# Sum numbers in a column
awk '/^[0-9]/ {sum+=$1} END {print sum}' numbers.txt

# Print lines between patterns
awk '/START/,/END/' file.txt

# Validate and process CSV
awk -F',' '$2 ~ /^[0-9]+$/ {print $1, $2}' data.csv
```

---

## Common Regex Patterns

### Email Validation

```bash
# Basic email pattern
^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$

# Usage
if [[ $email =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
    echo "Valid email"
fi
```

### IP Address

```bash
# IPv4 pattern
^([0-9]{1,3}\.){3}[0-9]{1,3}$

# More strict (0-255)
^((25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$
```

### URL Pattern

```bash
# Basic URL
^https?://[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}(/.*)?$

# With optional port
^https?://[a-zA-Z0-9.-]+(:[0-9]{1,5})?\.[a-zA-Z]{2,}(/.*)?$
```

### Date Patterns

```bash
# YYYY-MM-DD
^[0-9]{4}-[0-9]{2}-[0-9]{2}$

# DD/MM/YYYY
^[0-9]{2}/[0-9]{2}/[0-9]{4}$

# MM-DD-YYYY
^[0-9]{2}-[0-9]{2}-[0-9]{4}$
```

### Phone Numbers

```bash
# US format (123) 456-7890
^\([0-9]{3}\) [0-9]{3}-[0-9]{4}$

# International format
^\+?[0-9]{1,3}[-. ]?(\([0-9]{3}\)|[0-9]{3})[-. ]?[0-9]{3}[-. ]?[0-9]{4}$
```

---

## Practical Script Examples

### Log Parser

```bash
#!/bin/bash

LOG_FILE="/var/log/app.log"

# Extract error messages
echo "=== Errors ==="
grep -E "ERROR|FATAL" "$LOG_FILE"

# Count errors per hour
echo -e "\n=== Errors by Hour ==="
grep "ERROR" "$LOG_FILE" | sed -E 's/.*([0-9]{2}):[0-9]{2}:[0-9]{2}.*/\1/' | sort | uniq -c

# Extract IP addresses
echo -e "\n=== IP Addresses ==="
grep -oE "([0-9]{1,3}\.){3}[0-9]{1,3}" "$LOG_FILE" | sort -u

# Find failed login attempts
echo -e "\n=== Failed Logins ==="
grep -E "failed|failure|invalid" "$LOG_FILE" | grep -i "login\|auth"
```

### Email Validator

```bash
#!/bin/bash

validate_email() {
    local email=$1
    local regex='^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    
    if [[ $email =~ $regex ]]; then
        # Extract parts
        if [[ $email =~ ^([^@]+)@(.+)$ ]]; then
            local username="${BASH_REMATCH[1]}"
            local domain="${BASH_REMATCH[2]}"
            echo "Valid: User=$username, Domain=$domain"
            return 0
        fi
    else
        echo "Invalid email format"
        return 1
    fi
}

# Test
validate_email "user@example.com"
validate_email "invalid.email"
```

### Configuration File Parser

```bash
#!/bin/bash

parse_config() {
    local config_file=$1
    
    # Remove comments and empty lines
    sed 's/#.*$//' "$config_file" | grep -v "^$" | \
    while IFS='=' read -r key value; do
        # Trim whitespace
        key=$(echo "$key" | sed 's/^[[:space:]]*//;s/[[:space:]]*$//')
        value=$(echo "$value" | sed 's/^[[:space:]]*//;s/[[:space:]]*$//')
        
        # Validate key format (alphanumeric and underscore)
        if [[ $key =~ ^[a-zA-Z_][a-zA-Z0-9_]*$ ]]; then
            echo "export $key='$value'"
        fi
    done
}

# Usage
parse_config config.ini
```

---

## Best Practices

1. **Test regex patterns** incrementally
2. **Use extended regex** (`-E`) for readability
3. **Escape special characters** when matching literals
4. **Use anchors** (^, $) for precise matching
5. **Comment complex patterns** in scripts
6. **Use online regex testers** (regex101.com)
7. **Consider performance** for large files

---

## Summary

- Regular expressions provide powerful pattern matching
- Basic (BRE) vs Extended (ERE) - use `-E` for ERE
- Common tools: `grep`, `sed`, `awk`
- Use `[[ ]]` with `=~` for regex in Bash conditionals
- Master common patterns for emails, IPs, URLs, dates

---

## Practice Exercises

1. Write a script to validate different input formats (email, phone, IP)
2. Parse a log file to extract and summarize specific patterns
3. Create a sed script to clean and format text data
4. Build an awk script to analyze CSV data with validation

---

**Next Chapter:** [20 - Working with Processes](20-working-with-processes.md)
