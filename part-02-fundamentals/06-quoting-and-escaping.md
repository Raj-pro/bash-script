# Chapter 6: Quoting and Escaping

## Table of Contents
1. [Why Quoting Matters](#why-quoting-matters)
2. [Single Quotes](#single-quotes)
3. [Double Quotes](#double-quotes)
4. [Escape Characters](#escape-characters)
5. [ANSI-C Quoting](#ansi-c-quoting)
6. [Common Pitfalls](#common-pitfalls)
7. [Best Practices](#best-practices)
8. [Summary](#summary)

---

## Why Quoting Matters

Quoting in Bash controls how the shell interprets special characters and whitespace.

### Problems Without Proper Quoting

```bash
# Word splitting problem
file="my document.txt"
cat $file
# Error: tries to open "my" and "document.txt" separately

# Globbing problem
pattern="*.txt"
echo $pattern
# Expands to list of .txt files instead of literal "*.txt"

# Command substitution issue
msg="Current date: $(date)"
echo $msg
# Works, but could cause issues with special characters

# Variable injection
user_input="'; rm -rf /"
eval "echo $user_input"  # DANGEROUS!
```

---

## Single Quotes

### Behavior: Preserve Everything Literally

```bash
# Everything inside is literal
echo 'Hello $USER'
# Output: Hello $USER (no variable expansion)

echo 'Current dir: $(pwd)'
# Output: Current dir: $(pwd) (no command substitution)

echo 'Price: $100'
# Output: Price: $100

echo 'Two\nLines'
# Output: Two\nLines (no escape sequences)
```

### Use Cases for Single Quotes

```bash
# 1. Protect special characters
grep 'price: $[0-9]+' file.txt

# 2. Preserve exact strings
sql='SELECT * FROM users WHERE name = "O'\''Reilly"'

# 3. Prevent variable expansion
echo 'Use $HOME to reference home directory'

# 4. Regular expressions
pattern='^\d{3}-\d{2}-\d{4}$'

# 5. Literal text with special characters
message='Cost is $5 (including * & @ symbols)'
```

### Limitation: Can't Include Single Quote

```bash
# WRONG - syntax error
echo 'It's a beautiful day'

# SOLUTION 1: End quote, escape, start new quote
echo 'It'\''s a beautiful day'
# Breaks down to: 'It' + \' + 's a beautiful day'

# SOLUTION 2: Use double quotes
echo "It's a beautiful day"

# SOLUTION 3: Mix quotes
echo 'It'"'"'s a beautiful day'

# SOLUTION 4: Use $'...' syntax
echo $'It\'s a beautiful day'
```

---

## Double Quotes

### Behavior: Allow Variable and Command Expansion

```bash
# Variable expansion works
echo "Hello $USER"
# Output: Hello john

# Command substitution works
echo "Today is $(date +%A)"
# Output: Today is Friday

# Arithmetic expansion works
echo "5 + 3 = $((5 + 3))"
# Output: 5 + 3 = 8

# Preserves whitespace
text="Line 1
Line 2
Line 3"
echo "$text"
# Output: (preserves line breaks)
```

### Special Characters in Double Quotes

**These are still special:**
- `$` (variable expansion)
- `` ` `` (backtick command substitution)
- `\` (escape character)
- `!` (history expansion, in interactive shells)

**These become literal:**
- `*` (asterisk)
- `?` (question mark)
- `[ ]` (brackets)
- `~` (tilde)

```bash
# Globbing is prevented
echo "Files: *.txt"
# Output: Files: *.txt (literal)

# But variables expand
files="*.txt"
echo "Files: $files"
# Output: Files: (list of .txt files)

# Backslash escapes work
echo "Price: \$100"
# Output: Price: $100

echo "Path: C:\\Users\\Name"
# Output: Path: C:\Users\Name
```

### When to Use Double Quotes

```bash
# 1. Variable expansion needed
name="John"
echo "Hello, $name!"

# 2. Preserve whitespace
message="Line 1
         Line 2"
echo "$message"  # Preserves formatting

# 3. Command substitution
echo "Current directory: $(pwd)"

# 4. Prevent word splitting
filename="my file.txt"
cat "$filename"  # Works correctly

# 5. Array elements
files=("file 1.txt" "file 2.txt")
for file in "${files[@]}"; do
    echo "Processing: $file"
done
```

---

## Escape Characters

### Backslash (\)

**Escapes the next character, making it literal**

```bash
# Escape dollar sign
echo \$HOME
# Output: $HOME

# Escape newline (line continuation)
echo "This is a very long \
line split across multiple lines"

# Escape spaces
mkdir My\ Documents
cd My\ Documents

# Escape quotes
echo "He said \"Hello\""
# Output: He said "Hello"

echo 'It'\''s working'
# Output: It's working

# Escape special characters
echo "Cost: \$5.00"
echo "Path: C:\\Windows"
echo "Email: user\@domain.com"
```

### Common Escape Sequences

```bash
# In echo (requires -e flag)
echo -e "Line 1\nLine 2"        # Newline
echo -e "Col1\tCol2\tCol3"      # Tab
echo -e "Hello\rWorld"          # Carriage return
echo -e "Backspace:\b\b\bNow"   # Backspace
echo -e "\aBeep"                # Alert (beep)

# Without -e, they're literal
echo "Line 1\nLine 2"
# Output: Line 1\nLine 2
```

### Escape Sequences Table

| Sequence | Meaning |
|----------|---------|
| `\\` | Backslash |
| `\'` | Single quote |
| `\"` | Double quote |
| `\n` | Newline |
| `\t` | Tab |
| `\r` | Carriage return |
| `\b` | Backspace |
| `\a` | Alert (bell) |
| `\e` | Escape character |
| `\0NNN` | Octal value |
| `\xHH` | Hexadecimal value |

---

## ANSI-C Quoting

### $'...' Syntax

**Enables escape sequences without -e flag**

```bash
# Basic escape sequences
echo $'Line 1\nLine 2'
# Output:
# Line 1
# Line 2

echo $'Column1\tColumn2\tColumn3'
# Output: Column1    Column2    Column3

# Include single quotes
echo $'It\'s a beautiful day'
# Output: It's a beautiful day

# Unicode characters
echo $'\u2764 Love'
# Output: ❤ Love

echo $'\U0001F600 Smile'
# Output: 😀 Smile

# Hex values
echo $'\x48\x65\x6C\x6C\x6F'
# Output: Hello

# Colors
echo $'\e[31mRed text\e[0m'
# Output: Red text (in red)
```

### Color and Formatting

```bash
# Color codes using $'...'
RED=$'\e[31m'
GREEN=$'\e[32m'
YELLOW=$'\e[33m'
BLUE=$'\e[34m'
RESET=$'\e[0m'

echo "${RED}Error${RESET}"
echo "${GREEN}Success${RESET}"
echo "${YELLOW}Warning${RESET}"

# Bold, underline
BOLD=$'\e[1m'
UNDERLINE=$'\e[4m'

echo "${BOLD}Bold text${RESET}"
echo "${UNDERLINE}Underlined text${RESET}"
```

---

## Common Pitfalls

### 1. Unquoted Variables

```bash
# BAD
file=my file.txt
cat $file  # Error: tries to cat "my" and "file.txt"

# GOOD
cat "$file"

# BAD
files="*.txt"
rm $files  # Dangerous: expands to all .txt files

# GOOD (if you want literal)
rm "$files"
```

### 2. Word Splitting in Arrays

```bash
# BAD
files=("file 1.txt" "file 2.txt")
for file in ${files[@]}; do
    echo "$file"
done
# Output:
# file
# 1.txt
# file
# 2.txt

# GOOD
for file in "${files[@]}"; do
    echo "$file"
done
# Output:
# file 1.txt
# file 2.txt
```

### 3. Command Substitution Without Quotes

```bash
# BAD
output=$(cat file.txt)
echo $output  # Loses formatting, newlines become spaces

# GOOD
echo "$output"  # Preserves formatting
```

### 4. Mixing Quote Types

```bash
# WRONG
echo "It's $HOME's directory"  # Confusing

# BETTER
echo "It's ${HOME}'s directory"

# OR
echo 'It'\''s '"$HOME"''\''s directory'

# BEST (most readable)
echo "It's ${HOME}'s directory"
```

### 5. Glob Patterns

```bash
# Unquoted - globs
files=*.txt
echo $files  # Expands to file list

# Quoted - literal
files="*.txt"
echo "$files"  # Output: *.txt

# To actually expand:
files=(*.txt)
echo "${files[@]}"
```

---

## Best Practices

### 1. Default to Double Quotes

```bash
# Use double quotes as default for variables
echo "Value: $var"
cp "$source" "$dest"
rm "$filename"
```

### 2. Use Single Quotes for Literal Strings

```bash
# Regex patterns
grep '^Error:' logfile

# SQL queries
sql='SELECT * FROM users WHERE id = 1'

# Messages with special characters
echo 'Use $VAR to reference variable'
```

### 3. Quote All Variable References

```bash
# ALWAYS quote variables
if [ -f "$filename" ]; then
    cat "$filename"
fi

# Especially in loops
for file in "${files[@]}"; do
    process "$file"
done
```

### 4. Use ${} for Clarity

```bash
# Ambiguous
echo "$usernametxt"

# Clear
echo "${username}txt"
echo "${username}.txt"
```

### 5. ShellCheck for Validation

```bash
# Install ShellCheck
sudo apt install shellcheck

# Check script
shellcheck myscript.sh

# It will catch quoting issues like:
# SC2086: Double quote to prevent globbing and word splitting
# SC2068: Double quote array expansions
```

### 6. Consistent Style

```bash
# Good script example
#!/bin/bash

readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly CONFIG_FILE="${SCRIPT_DIR}/config.conf"

main() {
    local input="$1"
    local output="${2:-output.txt}"
    
    if [ ! -f "$input" ]; then
        echo "Error: File not found: $input" >&2
        exit 1
    fi
    
    process_file "$input" "$output"
}

process_file() {
    local source="$1"
    local dest="$2"
    
    echo "Processing: $source"
    cat "$source" > "$dest"
    echo "Saved to: $dest"
}

main "$@"
```

---

## Summary

### Quote Types Comparison

| Type | Syntax | Expands Variables | Expands Commands | Escape Sequences | Use Case |
|------|--------|-------------------|------------------|------------------|----------|
| **None** | `text` | ✅ Yes | ✅ Yes | ❌ No | Simple words |
| **Single** | `'text'` | ❌ No | ❌ No | ❌ No | Literal strings |
| **Double** | `"text"` | ✅ Yes | ✅ Yes | ⚠️ Some | Variables, whitespace |
| **ANSI-C** | `$'text'` | ❌ No | ❌ No | ✅ Yes | Escape sequences |

### Quick Decision Guide

```bash
# Use single quotes when:
'Literal text with special chars: $*?[]'

# Use double quotes when:
"Variables: $var or commands: $(cmd)"

# Use $'...' when:
$'Escape sequences:\n\t or unicode:\u2764'

# Use no quotes when:
# You know exactly what you're doing (rarely!)
```

### Key Takeaways

1. **Always quote variables** unless you know you need word splitting
2. **Single quotes** preserve everything literally
3. **Double quotes** allow variable/command expansion
4. **Escape with backslash** for individual characters
5. **$'...'** for escape sequences and unicode
6. **Use ShellCheck** to catch quoting errors

### Common Patterns

```bash
# Safe file operations
cp "$source" "$destination"
rm -f "$filename"

# Arrays
for item in "${array[@]}"; do
    echo "$item"
done

# Command substitution
result="$(command)"
echo "$result"

# Mixed quoting
echo "It's ${USER}'s file"

# Heredoc (alternative to complex quoting)
cat << 'EOF'
Everything here is literal
$VAR will not expand
EOF

cat << EOF
But here $VAR will expand
And $(date) will too
EOF
```

---

**Next Chapter:** [User Input and Output →](07-user-input-output.md)

---

*Practice Exercise: Create a script that handles filenames with spaces, uses both single and double quotes appropriately, and demonstrates the difference between quoted and unquoted variables.*
