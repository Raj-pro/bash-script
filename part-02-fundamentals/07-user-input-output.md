# Chapter 7: User Input and Output

## Table of Contents
1. [Reading User Input](#reading-user-input)
2. [Echo Command](#echo-command)
3. [Printf Command](#printf-command)
4. [Output Formatting](#output-formatting)
5. [Colors and Styling](#colors-and-styling)
6. [Interactive Prompts](#interactive-prompts)
7. [Best Practices](#best-practices)
8. [Summary](#summary)

---

## Reading User Input

### Basic read Command

```bash
# Simple input
read name
echo "Hello, $name"

# With prompt
read -p "Enter your name: " name
echo "Hello, $name"

# Multiple variables
read -p "Enter first and last name: " first last
echo "First: $first, Last: $last"

# Read entire line
read -p "Enter a sentence: " sentence
echo "You said: $sentence"
```

### read Command Options

```bash
# Silent input (passwords)
read -sp "Enter password: " password
echo  # New line after password input
echo "Password entered"

# Timeout (5 seconds)
if read -t 5 -p "Quick! Enter something: " response; then
    echo "You entered: $response"
else
    echo "Too slow!"
fi

# Read single character
read -n 1 -p "Press any key to continue..."
echo

# Read until delimiter
read -d ":" -p "Enter value (end with :): " value

# Default value if empty
read -p "Enter name [Default]: " name
name=${name:-Default}
echo "Name: $name"

# Into array
read -a words -p "Enter words: "
echo "First word: ${words[0]}"
echo "All words: ${words[@]}"

# From file descriptor
read -u 3 line  # Read from file descriptor 3
```

### Reading from Files

```bash
# Read file line by line
while read -r line; do
    echo "Line: $line"
done < file.txt

# Read with IFS (Internal Field Separator)
while IFS=: read -r username password uid gid; do
    echo "User: $username, UID: $uid"
done < /etc/passwd

# Read entire file into variable
content=$(< file.txt)
# or
content=$(cat file.txt)
```

### Advanced Input Handling

```bash
# Read with prompt and validation
while true; do
    read -p "Enter age (18-100): " age
    if [[ "$age" =~ ^[0-9]+$ ]] && [ "$age" -ge 18 ] && [ "$age" -le 100 ]; then
        break
    else
        echo "Invalid age. Please try again."
    fi
done

# Read yes/no confirmation
read -p "Are you sure? (y/n): " confirm
case "$confirm" in
    [yY]|[yY][eE][sS]) echo "Confirmed";;
    [nN]|[nN][oO]) echo "Cancelled"; exit 1;;
    *) echo "Invalid response";;
esac

# Read menu selection
echo "1) Option 1"
echo "2) Option 2"
echo "3) Option 3"
read -p "Select option: " choice
```

---

## Echo Command

### Basic Usage

```bash
# Simple output
echo "Hello, World!"

# Multiple arguments
echo "Hello" "World"
# Output: Hello World

# Variables
name="John"
echo "Hello, $name"

# Without newline (-n)
echo -n "Loading..."
sleep 2
echo " Done!"
# Output: Loading... Done!

# Interpret escape sequences (-e)
echo -e "Line 1\nLine 2"
echo -e "Col1\tCol2\tCol3"
```

### Escape Sequences with echo -e

```bash
# Newline
echo -e "First\nSecond\nThird"

# Tab
echo -e "Name:\tJohn\nAge:\t25"

# Carriage return
echo -e "Hello\rWorld"  # World overwrites Hello

# Backspace
echo -e "12345\b\b67"  # Output: 12367

# Vertical tab
echo -e "Line1\vLine2"

# Alert/Bell
echo -e "\aBeep!"

# Color codes
echo -e "\e[31mRed Text\e[0m"
```

### Echo vs printf

```bash
# echo is simpler for basic output
echo "Simple message"

# printf is better for formatting
printf "Name: %s, Age: %d\n" "John" 25

# echo has portability issues
# Some systems interpret \n, others don't
echo "Line1\nLine2"  # Behavior varies

# printf is more consistent
printf "Line1\nLine2\n"
```

---

## Printf Command

### Basic Syntax

```bash
# Format: printf "format" arguments

# Simple string
printf "Hello, World!\n"

# With variable
name="John"
printf "Hello, %s!\n" "$name"

# Multiple variables
printf "Name: %s, Age: %d\n" "John" 25

# Without newline
printf "Loading..."
sleep 2
printf " Done!\n"
```

### Format Specifiers

```bash
# %s - String
printf "Name: %s\n" "John"

# %d - Decimal integer
printf "Age: %d\n" 25

# %f - Floating point
printf "Price: %.2f\n" 19.99

# %x - Hexadecimal
printf "Hex: %x\n" 255  # Output: ff

# %o - Octal
printf "Octal: %o\n" 8  # Output: 10

# %% - Literal %
printf "Discount: 50%%\n"

# %c - Single character
printf "Grade: %c\n" A
```

### Field Width and Alignment

```bash
# Right-aligned (default)
printf "%10s\n" "Hello"
# Output: "     Hello"

# Left-aligned
printf "%-10s\n" "Hello"
# Output: "Hello     "

# Numbers with width
printf "%5d\n" 42
# Output: "   42"

# Zero-padding
printf "%05d\n" 42
# Output: "00042"

# Float precision
printf "%.2f\n" 3.14159
# Output: "3.14"

printf "%8.2f\n" 3.14159
# Output: "    3.14"
```

### Formatted Tables

```bash
# Create table headers
printf "%-15s %-10s %10s\n" "Name" "Age" "Score"
printf "%-15s %-10s %10s\n" "---------------" "----------" "----------"

# Table data
printf "%-15s %-10d %10.2f\n" "John Doe" 25 95.5
printf "%-15s %-10d %10.2f\n" "Jane Smith" 30 87.3
printf "%-15s %-10d %10.2f\n" "Bob Johnson" 22 92.8

# Output:
# Name            Age              Score
# --------------- ----------  ----------
# John Doe        25               95.50
# Jane Smith      30               87.30
# Bob Johnson     22               92.80
```

### Printf in Loops

```bash
# Format multiple items
users=("Alice:25" "Bob:30" "Charlie:28")

printf "%-15s %s\n" "Name" "Age"
printf "%-15s %s\n" "----" "---"

for user in "${users[@]}"; do
    IFS=: read -r name age <<< "$user"
    printf "%-15s %d\n" "$name" "$age"
done
```

---

## Output Formatting

### Columnar Output

```bash
# Using column command
echo -e "Name\tAge\tCity"
echo -e "John\t25\tNY"
echo -e "Jane\t30\tLA"
} | column -t -s $'\t'

# Output:
# Name  Age  City
# John  25   NY
# Jane  30   LA
```

### Boxes and Borders

```bash
# Simple box
echo "┌─────────────────┐"
echo "│   Hello World   │"
echo "└─────────────────┘"

# Function to create box
print_box() {
    local text="$1"
    local len=${#text}
    local border=$(printf '%*s' $((len + 4)) | tr ' ' '─')
    
    echo "┌${border}┐"
    echo "│  $text  │"
    echo "└${border}┘"
}

print_box "Welcome"
```

### Progress Indicators

```bash
# Simple progress bar
show_progress() {
    local current=$1
    local total=$2
    local width=50
    local percentage=$((current * 100 / total))
    local filled=$((current * width / total))
    
    printf "\r["
    printf "%${filled}s" | tr ' ' '='
    printf "%$((width - filled))s" | tr ' ' ' '
    printf "] %3d%%" "$percentage"
}

# Usage
for i in {1..100}; do
    show_progress $i 100
    sleep 0.05
done
echo

# Spinner
spinner() {
    local pid=$1
    local delay=0.1
    local spinstr='|/-\'
    
    while kill -0 $pid 2>/dev/null; do
        local temp=${spinstr#?}
        printf " [%c]  " "$spinstr"
        spinstr=$temp${spinstr%"$temp"}
        sleep $delay
        printf "\b\b\b\b\b\b"
    done
    printf "    \b\b\b\b"
}

# Usage
long_task &
spinner $!
```

---

## Colors and Styling

### ANSI Color Codes

```bash
# Basic colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[0;33m'
BLUE='\033[0;34m'
MAGENTA='\033[0;35m'
CYAN='\033[0;36m'
WHITE='\033[0;37m'
RESET='\033[0m'

# Usage
echo -e "${RED}Error${RESET}"
echo -e "${GREEN}Success${RESET}"
echo -e "${YELLOW}Warning${RESET}"

# Printf
printf "${BLUE}Info:${RESET} This is information\n"
```

### Text Formatting

```bash
# Styles
BOLD='\033[1m'
DIM='\033[2m'
UNDERLINE='\033[4m'
BLINK='\033[5m'
REVERSE='\033[7m'
HIDDEN='\033[8m'
RESET='\033[0m'

# Usage
echo -e "${BOLD}Bold text${RESET}"
echo -e "${UNDERLINE}Underlined${RESET}"
echo -e "${REVERSE}Reversed${RESET}"

# Combine
echo -e "${BOLD}${RED}Bold Red${RESET}"
echo -e "${UNDERLINE}${GREEN}Underlined Green${RESET}"
```

### Background Colors

```bash
# Background colors
BG_BLACK='\033[40m'
BG_RED='\033[41m'
BG_GREEN='\033[42m'
BG_YELLOW='\033[43m'
BG_BLUE='\033[44m'
BG_MAGENTA='\033[45m'
BG_CYAN='\033[46m'
BG_WHITE='\033[47m'

# Usage
echo -e "${BG_RED}${WHITE}White on Red${RESET}"
echo -e "${BG_GREEN}${BLACK}Black on Green${RESET}"
```

### 256 Colors

```bash
# 256 color support
# Foreground: \033[38;5;COLORm
# Background: \033[48;5;COLORm

# Examples
echo -e "\033[38;5;196mBright Red\033[0m"
echo -e "\033[38;5;51mBright Cyan\033[0m"
echo -e "\033[48;5;208mOrange Background\033[0m"

# Color palette display
for i in {0..255}; do
    printf "\033[38;5;%dm%3d " "$i" "$i"
    [ $(((i + 1) % 16)) -eq 0 ] && echo
done
echo -e "\033[0m"
```

### Helper Functions

```bash
# Color functions
error() {
    echo -e "${RED}ERROR:${RESET} $*" >&2
}

success() {
    echo -e "${GREEN}SUCCESS:${RESET} $*"
}

warning() {
    echo -e "${YELLOW}WARNING:${RESET} $*"
}

info() {
    echo -e "${BLUE}INFO:${RESET} $*"
}

# Usage
error "File not found"
success "Operation completed"
warning "Low disk space"
info "Starting process"
```

---

## Interactive Prompts

### Select Menu

```bash
# Built-in select
echo "Choose an option:"
options=("Install" "Update" "Remove" "Quit")

select opt in "${options[@]}"; do
    case $opt in
        "Install") echo "Installing..."; break;;
        "Update") echo "Updating..."; break;;
        "Remove") echo "Removing..."; break;;
        "Quit") echo "Goodbye!"; exit;;
        *) echo "Invalid option";;
    esac
done
```

### Custom Menu

```bash
show_menu() {
    clear
    echo "===================="
    echo "   MAIN MENU"
    echo "===================="
    echo "1. Option 1"
    echo "2. Option 2"
    echo "3. Option 3"
    echo "4. Exit"
    echo "===================="
}

while true; do
    show_menu
    read -p "Enter choice [1-4]: " choice
    
    case $choice in
        1) echo "You selected Option 1"; read -p "Press Enter to continue...";;
        2) echo "You selected Option 2"; read -p "Press Enter to continue...";;
        3) echo "You selected Option 3"; read -p "Press Enter to continue...";;
        4) echo "Goodbye!"; exit 0;;
        *) echo "Invalid option"; sleep 2;;
    esac
done
```

### Confirmation Prompts

```bash
# Simple yes/no
confirm() {
    read -p "$1 (y/n): " -n 1 -r
    echo
    [[ $REPLY =~ ^[Yy]$ ]]
}

if confirm "Delete file?"; then
    echo "Deleting..."
else
    echo "Cancelled"
fi

# Default yes
confirm_default_yes() {
    read -p "$1 [Y/n]: " -r
    [[ ! $REPLY =~ ^[Nn]$ ]]
}

# Default no
confirm_default_no() {
    read -p "$1 [y/N]: " -r
    [[ $REPLY =~ ^[Yy]$ ]]
}
```

---

## Best Practices

### 1. Use printf for Complex Formatting

```bash
# BAD
echo "Name: $name, Age: $age, Score: $score"

# GOOD
printf "Name: %-15s Age: %3d Score: %6.2f\n" "$name" "$age" "$score"
```

### 2. Quote Variables in Output

```bash
# Always quote to prevent word splitting
echo "File: $filename"
printf "Path: %s\n" "$path"
```

### 3. Redirect Errors Appropriately

```bash
# Success messages to stdout
echo "Operation successful"

# Error messages to stderr
echo "Error occurred" >&2
```

### 4. Use Functions for Consistency

```bash
# Consistent logging
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*"
}

log "Application started"
log "Processing file: $filename"
```

### 5. Handle Special Characters

```bash
# Use printf for special characters
printf "%s\n" "$text"  # Safer than echo

# Or use echo with --
echo -- "$text"
```

---

## Summary

### Input Methods

```bash
read var                    # Basic input
read -p "Prompt: " var      # With prompt
read -sp "Pass: " var       # Silent
read -t 5 var               # Timeout
read -n 1 var               # Single char
read -a arr                 # Into array
```

### Output Commands

```bash
echo "text"                 # Simple output
echo -n "text"              # No newline
echo -e "text\n"            # Escape sequences
printf "fmt" args           # Formatted output
```

### Format Specifiers

```bash
%s          # String
%d          # Integer
%f          # Float
%.2f        # Float with 2 decimals
%10s        # Right-aligned, width 10
%-10s       # Left-aligned, width 10
%05d        # Zero-padded number
```

### Color Codes

```bash
\033[0;31m  # Red
\033[0;32m  # Green
\033[0;33m  # Yellow
\033[0;34m  # Blue
\033[0m     # Reset
\033[1m     # Bold
\033[4m     # Underline
```

### Common Patterns

```bash
# Colored output
error() { echo -e "\033[0;31mERROR:\033[0m $*" >&2; }

# Formatted table
printf "%-15s %10s\n" "Name" "Value"

# Progress
printf "\r[%3d%%]" "$percentage"

# Read with validation
while ! [[ $input =~ ^[0-9]+$ ]]; do
    read -p "Enter number: " input
done
```

---

**Next Chapter:** [Command-Line Arguments →](08-command-line-arguments.md)

---

*Practice Exercise: Create an interactive script with a menu, colored output, and formatted table display. Add input validation and confirmation prompts.*
