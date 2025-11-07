# Chapter 11: Case Statements

## Table of Contents
1. [Introduction to Case Statements](#introduction-to-case-statements)
2. [Basic Syntax](#basic-syntax)
3. [Pattern Matching](#pattern-matching)
4. [Multiple Patterns](#multiple-patterns)
5. [Menu Systems](#menu-systems)
6. [Practical Applications](#practical-applications)
7. [Best Practices](#best-practices)
8. [Summary](#summary)

---

## Introduction to Case Statements

The `case` statement is a cleaner alternative to multiple `if-elif-else` chains when comparing a single variable against multiple values.

### Why Use Case?

```bash
# Without case (messy)
if [ "$choice" = "1" ]; then
    start_service
elif [ "$choice" = "2" ]; then
    stop_service
elif [ "$choice" = "3" ]; then
    restart_service
elif [ "$choice" = "4" ]; then
    status_service
else
    echo "Invalid"
fi

# With case (clean)
case "$choice" in
    1) start_service;;
    2) stop_service;;
    3) restart_service;;
    4) status_service;;
    *) echo "Invalid";;
esac
```

---

## Basic Syntax

### Structure

```bash
case EXPRESSION in
    PATTERN1)
        commands
        ;;
    PATTERN2)
        commands
        ;;
    *)
        default_commands
        ;;
esac
```

### Simple Examples

```bash
# Basic case statement
fruit="apple"

case "$fruit" in
    apple)
        echo "It's an apple"
        ;;
    banana)
        echo "It's a banana"
        ;;
    orange)
        echo "It's an orange"
        ;;
    *)
        echo "Unknown fruit"
        ;;
esac

# With numbers
read -p "Enter 1-3: " num

case "$num" in
    1)
        echo "One"
        ;;
    2)
        echo "Two"
        ;;
    3)
        echo "Three"
        ;;
    *)
        echo "Invalid number"
        ;;
esac
```

### Single Line Format

```bash
# Compact syntax
case "$var" in
    value1) command1;;
    value2) command2;;
    *) default_command;;
esac

# Example
case "$1" in
    start) systemctl start myapp;;
    stop) systemctl stop myapp;;
    restart) systemctl restart myapp;;
    *) echo "Usage: $0 {start|stop|restart}";;
esac
```

---

## Pattern Matching

### Wildcard Patterns

```bash
# Asterisk (*)
case "$filename" in
    *.txt)
        echo "Text file"
        ;;
    *.jpg|*.png)
        echo "Image file"
        ;;
    *.tar.gz)
        echo "Compressed archive"
        ;;
    *)
        echo "Unknown file type"
        ;;
esac

# Question mark (?)
case "$var" in
    ?)
        echo "Single character"
        ;;
    ??)
        echo "Two characters"
        ;;
    *)
        echo "More than two characters"
        ;;
esac
```

### Character Classes

```bash
# Bracket expressions
case "$char" in
    [a-z])
        echo "Lowercase letter"
        ;;
    [A-Z])
        echo "Uppercase letter"
        ;;
    [0-9])
        echo "Digit"
        ;;
    *)
        echo "Special character"
        ;;
esac

# Multiple character ranges
case "$input" in
    [a-zA-Z])
        echo "Letter"
        ;;
    [0-9])
        echo "Number"
        ;;
    [!a-zA-Z0-9])
        echo "Special character"
        ;;
esac
```

### Complex Patterns

```bash
# Starts with
case "$string" in
    user*)
        echo "Starts with 'user'"
        ;;
    admin*)
        echo "Starts with 'admin'"
        ;;
esac

# Ends with
case "$file" in
    *.log)
        echo "Log file"
        ;;
    *.conf)
        echo "Config file"
        ;;
esac

# Contains (requires extended globbing)
shopt -s extglob

case "$path" in
    *"/bin/"*)
        echo "Path contains /bin/"
        ;;
    *"/etc/"*)
        echo "Path contains /etc/"
        ;;
esac
```

---

## Multiple Patterns

### OR Patterns (|)

```bash
# Multiple values for same action
case "$answer" in
    y|Y|yes|Yes|YES)
        echo "Confirmed"
        ;;
    n|N|no|No|NO)
        echo "Cancelled"
        ;;
    *)
        echo "Invalid response"
        ;;
esac

# File extensions
case "$filename" in
    *.jpg|*.jpeg|*.png|*.gif)
        echo "Image file"
        process_image "$filename"
        ;;
    *.mp4|*.avi|*.mkv)
        echo "Video file"
        process_video "$filename"
        ;;
    *.txt|*.md|*.doc)
        echo "Document"
        process_document "$filename"
        ;;
esac
```

### Fall-through with ;;&

```bash
# Bash 4+ feature: continue matching
case "$file" in
    *.txt)
        echo "Text file"
        ;;&  # Continue to next pattern
    *.log)
        echo "Could be a log file"
        ;;
    *)
        echo "Unknown"
        ;;
esac

# Example with multiple matches
number=5
case "$number" in
    [0-9])
        echo "Single digit"
        ;;&
    [02468])
        echo "Even number"
        ;;
    [13579])
        echo "Odd number"
        ;;
esac
```

---

## Menu Systems

### Simple Menu

```bash
#!/bin/bash

while true; do
    echo ""
    echo "===== Main Menu ====="
    echo "1. Show date"
    echo "2. Show calendar"
    echo "3. Show disk usage"
    echo "4. Exit"
    echo "===================="
    read -p "Enter choice [1-4]: " choice
    
    case "$choice" in
        1)
            date
            ;;
        2)
            cal
            ;;
        3)
            df -h
            ;;
        4)
            echo "Goodbye!"
            exit 0
            ;;
        *)
            echo "Invalid choice. Please try again."
            ;;
    esac
    
    read -p "Press Enter to continue..."
done
```

### Advanced Menu

```bash
#!/bin/bash

show_menu() {
    clear
    echo "================================"
    echo "   System Management Menu"
    echo "================================"
    echo "1. User Management"
    echo "2. Service Control"
    echo "3. System Information"
    echo "4. Network Tools"
    echo "5. Exit"
    echo "================================"
}

user_menu() {
    echo "1. List users"
    echo "2. Add user"
    echo "3. Delete user"
    read -p "Choice: " subchoice
    
    case "$subchoice" in
        1) cat /etc/passwd | cut -d: -f1;;
        2) read -p "Username: " user; sudo useradd "$user";;
        3) read -p "Username: " user; sudo userdel "$user";;
        *) echo "Invalid choice";;
    esac
}

service_menu() {
    read -p "Service name: " service
    echo "1. Start"
    echo "2. Stop"
    echo "3. Restart"
    echo "4. Status"
    read -p "Choice: " subchoice
    
    case "$subchoice" in
        1) sudo systemctl start "$service";;
        2) sudo systemctl stop "$service";;
        3) sudo systemctl restart "$service";;
        4) sudo systemctl status "$service";;
        *) echo "Invalid choice";;
    esac
}

while true; do
    show_menu
    read -p "Enter choice [1-5]: " choice
    
    case "$choice" in
        1) user_menu;;
        2) service_menu;;
        3) uname -a; uptime; free -h;;
        4) ip addr; netstat -tuln;;
        5) echo "Exiting..."; exit 0;;
        *) echo "Invalid choice";;
    esac
    
    echo ""
    read -p "Press Enter to continue..."
done
```

### Select Menu (Built-in)

```bash
#!/bin/bash

PS3="Select operation: "
options=("Install" "Update" "Remove" "List" "Quit")

select opt in "${options[@]}"; do
    case "$opt" in
        "Install")
            read -p "Package name: " pkg
            sudo apt install "$pkg"
            ;;
        "Update")
            sudo apt update && sudo apt upgrade
            ;;
        "Remove")
            read -p "Package name: " pkg
            sudo apt remove "$pkg"
            ;;
        "List")
            dpkg -l
            ;;
        "Quit")
            break
            ;;
        *)
            echo "Invalid option $REPLY"
            ;;
    esac
done
```

---

## Practical Applications

### File Type Handler

```bash
#!/bin/bash

handle_file() {
    local file="$1"
    
    case "$file" in
        *.tar.gz|*.tgz)
            echo "Extracting tar.gz: $file"
            tar -xzf "$file"
            ;;
        *.tar.bz2|*.tbz2)
            echo "Extracting tar.bz2: $file"
            tar -xjf "$file"
            ;;
        *.zip)
            echo "Extracting zip: $file"
            unzip "$file"
            ;;
        *.rar)
            echo "Extracting rar: $file"
            unrar x "$file"
            ;;
        *.jpg|*.jpeg|*.png)
            echo "Displaying image: $file"
            display "$file"
            ;;
        *.mp3|*.wav)
            echo "Playing audio: $file"
            mpg123 "$file"
            ;;
        *.pdf)
            echo "Opening PDF: $file"
            evince "$file"
            ;;
        *)
            echo "Unknown file type: $file"
            file "$file"
            ;;
    esac
}

# Process all arguments
for file in "$@"; do
    handle_file "$file"
done
```

### Service Manager

```bash
#!/bin/bash

if [ $# -ne 2 ]; then
    echo "Usage: $0 <service> <action>"
    exit 1
fi

service="$1"
action="$2"

case "$action" in
    start|stop|restart|reload)
        echo "Executing: systemctl $action $service"
        sudo systemctl "$action" "$service"
        ;;
    status)
        systemctl status "$service"
        ;;
    enable)
        sudo systemctl enable "$service"
        echo "$service enabled at boot"
        ;;
    disable)
        sudo systemctl disable "$service"
        echo "$service disabled at boot"
        ;;
    log|logs)
        journalctl -u "$service" -n 50
        ;;
    *)
        echo "Invalid action: $action"
        echo "Valid actions: start, stop, restart, reload, status, enable, disable, log"
        exit 1
        ;;
esac
```

### Environment Detector

```bash
#!/bin/bash

# Detect operating system
detect_os() {
    case "$(uname -s)" in
        Linux*)
            if [ -f /etc/debian_version ]; then
                echo "Debian/Ubuntu"
            elif [ -f /etc/redhat-release ]; then
                echo "RedHat/CentOS"
            elif [ -f /etc/arch-release ]; then
                echo "Arch Linux"
            else
                echo "Linux (Unknown)"
            fi
            ;;
        Darwin*)
            echo "macOS"
            ;;
        CYGWIN*|MINGW*|MSYS*)
            echo "Windows"
            ;;
        *)
            echo "Unknown OS"
            ;;
    esac
}

OS=$(detect_os)

case "$OS" in
    "Debian/Ubuntu")
        PKG_MANAGER="apt"
        INSTALL_CMD="sudo apt install -y"
        ;;
    "RedHat/CentOS")
        PKG_MANAGER="yum"
        INSTALL_CMD="sudo yum install -y"
        ;;
    "Arch Linux")
        PKG_MANAGER="pacman"
        INSTALL_CMD="sudo pacman -S --noconfirm"
        ;;
    "macOS")
        PKG_MANAGER="brew"
        INSTALL_CMD="brew install"
        ;;
    *)
        echo "Unsupported OS: $OS"
        exit 1
        ;;
esac

echo "Detected OS: $OS"
echo "Package Manager: $PKG_MANAGER"
```

### CLI Tool Wrapper

```bash
#!/bin/bash
# git-helper.sh - Git command wrapper

command="$1"
shift

case "$command" in
    commit|c)
        message="$*"
        git add -A
        git commit -m "$message"
        ;;
    push|p)
        git push origin "$(git branch --show-current)"
        ;;
    pull|pl)
        git pull origin "$(git branch --show-current)"
        ;;
    status|s)
        git status
        ;;
    branch|b)
        case "$1" in
            create|c)
                git checkout -b "$2"
                ;;
            delete|d)
                git branch -d "$2"
                ;;
            list|l)
                git branch
                ;;
            *)
                git branch
                ;;
        esac
        ;;
    log|l)
        git log --oneline --graph --decorate -n 10
        ;;
    diff|d)
        git diff
        ;;
    *)
        echo "Unknown command: $command"
        echo "Available commands:"
        echo "  commit|c <message>  - Commit all changes"
        echo "  push|p              - Push to remote"
        echo "  pull|pl             - Pull from remote"
        echo "  status|s            - Show status"
        echo "  branch|b            - Branch operations"
        echo "  log|l               - Show log"
        echo "  diff|d              - Show diff"
        exit 1
        ;;
esac
```

---

## Best Practices

### 1. Always Include Default Case (*)

```bash
# Always handle unexpected values
case "$var" in
    expected1) action1;;
    expected2) action2;;
    *) echo "Unexpected value: $var";;  # Always include!
esac
```

### 2. Use Descriptive Patterns

```bash
# BAD - unclear
case "$1" in
    1) do_something;;
    2) do_other;;
esac

# GOOD - clear intent
case "$action" in
    start) start_service;;
    stop) stop_service;;
esac
```

### 3. Group Related Patterns

```bash
# Group similar cases
case "$file" in
    # Images
    *.jpg|*.jpeg|*.png|*.gif)
        process_image
        ;;
    # Documents
    *.doc|*.docx|*.pdf)
        process_document
        ;;
    # Archives
    *.zip|*.tar.gz|*.rar)
        extract_archive
        ;;
esac
```

### 4. Quote Variables

```bash
# Always quote the variable
case "$var" in  # GOOD
    pattern) action;;
esac

case $var in    # BAD - can cause issues
    pattern) action;;
esac
```

### 5. Use Functions for Complex Actions

```bash
# Keep case statements clean
case "$action" in
    complex1)
        handle_complex_action_1
        ;;
    complex2)
        handle_complex_action_2
        ;;
esac

handle_complex_action_1() {
    # Many lines of code here
    echo "Doing complex stuff"
}
```

---

## Summary

### Basic Syntax

```bash
case "$variable" in
    pattern1)
        commands
        ;;
    pattern2|pattern3)
        commands
        ;;
    *)
        default_commands
        ;;
esac
```

### Pattern Types

| Pattern | Matches | Example |
|---------|---------|---------|
| `literal` | Exact match | `start`, `stop` |
| `*` | Any string | `*.txt`, `user*` |
| `?` | Single char | `?.log`, `file?` |
| `[abc]` | Any char in set | `[0-9]`, `[a-z]` |
| `[!abc]` | Not in set | `[!0-9]` |
| `a\|b` | Multiple patterns | `*.txt\|*.log` |

### Common Patterns

```bash
# File extensions
case "$file" in
    *.txt) echo "Text";;
    *.jpg|*.png) echo "Image";;
esac

# Yes/No
case "$answer" in
    [yY]|[yY][eE][sS]) echo "Yes";;
    [nN]|[nN][oO]) echo "No";;
esac

# Menu choice
case "$choice" in
    1) action1;;
    2) action2;;
    *) echo "Invalid";;
esac

# Service actions
case "$action" in
    start|stop|restart) systemctl "$action" "$service";;
    *) echo "Usage: {start|stop|restart}";;
esac
```

### When to Use Case vs If

**Use case when:**
- Comparing one variable against many values
- Pattern matching needed
- Building menus
- Cleaner, more readable code

**Use if when:**
- Multiple variables to check
- Complex conditions with AND/OR
- Numeric comparisons
- Command execution results

---

**Next Chapter:** [Bash Functions →](../part-04-functions-modular/12-bash-functions.md)

---

*Practice Exercise: Create a complete system administration menu using case statements, including file type detection, service management, and user interaction with pattern matching.*
