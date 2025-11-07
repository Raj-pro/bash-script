# Chapter 10: Loops in Bash

## Table of Contents
1. [Introduction to Loops](#introduction-to-loops)
2. [for Loop](#for-loop)
3. [while Loop](#while-loop)
4. [until Loop](#until-loop)
5. [Loop Control](#loop-control)
6. [Nested Loops](#nested-loops)
7. [Practical Examples](#practical-examples)
8. [Summary](#summary)

---

## Introduction to Loops

Loops allow you to execute commands repeatedly until a condition is met.

**Three types of loops in Bash:**
- `for` - Iterate over a list
- `while` - Execute while condition is true
- `until` - Execute until condition is true

---

## for Loop

### Basic for Loop

```bash
# Iterate over list
for item in apple banana cherry; do
    echo "Fruit: $item"
done

# Output:
# Fruit: apple
# Fruit: banana
# Fruit: cherry

# With variables
fruits="apple banana cherry"
for fruit in $fruits; do
    echo "$fruit"
done

# Array iteration
files=(file1.txt file2.txt file3.txt)
for file in "${files[@]}"; do
    echo "Processing: $file"
done
```

### C-style for Loop

```bash
# Classic C syntax
for ((i=0; i<10; i++)); do
    echo "Number: $i"
done

# Countdown
for ((i=10; i>=0; i--)); do
    echo "$i"
    sleep 1
done
echo "Liftoff!"

# Step by 2
for ((i=0; i<=20; i+=2)); do
    echo "Even: $i"
done
```

### Brace Expansion

```bash
# Range 1-10
for i in {1..10}; do
    echo "Number: $i"
done

# Range with step (Bash 4+)
for i in {0..100..10}; do
    echo "$i"
done
# Output: 0 10 20 30 40 50 60 70 80 90 100

# Letters
for letter in {a..z}; do
    echo "$letter"
done

# Zero-padded
for num in {001..010}; do
    echo "File_$num.txt"
done
```

### Iterating Over Files

```bash
# All files in directory
for file in *; do
    echo "Found: $file"
done

# Specific file type
for txtfile in *.txt; do
    echo "Text file: $txtfile"
done

# Multiple patterns
for file in *.txt *.log *.csv; do
    [ -f "$file" ] && echo "Processing: $file"
done

# Recursive (with globstar)
shopt -s globstar
for file in **/*.txt; do
    echo "$file"
done

# Using find
find . -name "*.txt" | while read -r file; do
    echo "Found: $file"
done
```

### Command Substitution

```bash
# Iterate over command output
for user in $(cat /etc/passwd | cut -d: -f1); do
    echo "User: $user"
done

# Better: use while read
while IFS=: read -r username _; do
    echo "User: $username"
done < /etc/passwd

# Process list
for pid in $(pgrep firefox); do
    echo "Firefox PID: $pid"
done

# Date ranges
for day in $(seq 1 31); do
    echo "Day $day"
done
```

---

## while Loop

### Basic while Loop

```bash
# Counter
count=1
while [ $count -le 5 ]; do
    echo "Count: $count"
    ((count++))
done

# Infinite loop (Ctrl+C to stop)
while true; do
    echo "Running..."
    sleep 1
done

# While with condition
while [ -f /tmp/running ]; do
    echo "Service running"
    sleep 5
done
```

### Reading Files Line by Line

```bash
# Read file
while read -r line; do
    echo "Line: $line"
done < file.txt

# Read with field separator
while IFS=: read -r user pass uid gid info home shell; do
    echo "User: $user, Shell: $shell"
done < /etc/passwd

# Read CSV
while IFS=, read -r name age city; do
    echo "Name: $name, Age: $age, City: $city"
done < data.csv

# Skip header
{
    read  # Skip first line
    while IFS=, read -r name age city; do
        echo "$name is $age years old"
    done
} < data.csv
```

### while with Commands

```bash
# While command succeeds
while ping -c 1 google.com > /dev/null 2>&1; do
    echo "Internet connected"
    sleep 5
done
echo "Connection lost"

# While process running
while pgrep -x "myapp" > /dev/null; do
    echo "App is running"
    sleep 10
done

# While directory not empty
while [ -n "$(ls -A /tmp/queue)" ]; do
    echo "Processing queue..."
    sleep 1
done
```

### Menu Example

```bash
#!/bin/bash

choice=""
while [ "$choice" != "4" ]; do
    echo ""
    echo "1. Show date"
    echo "2. Show users"
    echo "3. Show uptime"
    echo "4. Exit"
    read -p "Enter choice: " choice
    
    case $choice in
        1) date;;
        2) who;;
        3) uptime;;
        4) echo "Goodbye!";;
        *) echo "Invalid choice";;
    esac
done
```

---

## until Loop

### Basic until Loop

```bash
# until is opposite of while
count=1
until [ $count -gt 5 ]; do
    echo "Count: $count"
    ((count++))
done

# Wait until file exists
until [ -f /tmp/ready ]; do
    echo "Waiting for file..."
    sleep 1
done
echo "File found!"

# Wait for service
until curl -s http://localhost:8080 > /dev/null; do
    echo "Waiting for service..."
    sleep 2
done
echo "Service is up!"
```

### Practical until Examples

```bash
# Wait for port to be available
until netstat -tuln | grep -q ":3306 "; do
    echo "Waiting for MySQL..."
    sleep 2
done

# Retry command until success
attempts=0
until mysql -e "SELECT 1" > /dev/null 2>&1; do
    ((attempts++))
    if [ $attempts -ge 10 ]; then
        echo "Failed after 10 attempts"
        exit 1
    fi
    echo "Attempt $attempts failed, retrying..."
    sleep 3
done

# Wait for variable to be set
until [ -n "$READY" ]; do
    source config.sh
    sleep 1
done
```

---

## Loop Control

### break - Exit Loop

```bash
# Exit loop early
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        echo "Breaking at $i"
        break
    fi
    echo "Number: $i"
done
echo "Loop ended"

# Break from while
count=1
while true; do
    echo "Count: $count"
    if [ $count -ge 5 ]; then
        break
    fi
    ((count++))
done

# Search and break
for file in *.txt; do
    if grep -q "ERROR" "$file"; then
        echo "Found error in: $file"
        break
    fi
done
```

### continue - Skip Iteration

```bash
# Skip even numbers
for i in {1..10}; do
    if [ $((i % 2)) -eq 0 ]; then
        continue
    fi
    echo "Odd: $i"
done

# Skip hidden files
for file in *; do
    [[ $file == .* ]] && continue
    echo "Processing: $file"
done

# Skip empty lines
while read -r line; do
    [ -z "$line" ] && continue
    echo "Line: $line"
done < file.txt
```

### break vs continue

```bash
#!/bin/bash

echo "=== break example ==="
for i in {1..5}; do
    echo "Outer: $i"
    for j in {1..5}; do
        if [ $j -eq 3 ]; then
            echo "  Breaking inner loop"
            break  # Exits inner loop only
        fi
        echo "  Inner: $j"
    done
done

echo ""
echo "=== continue example ==="
for i in {1..5}; do
    if [ $i -eq 3 ]; then
        continue  # Skips to next iteration
    fi
    echo "Number: $i"
done
```

### break with Levels

```bash
# Break out of nested loops
for i in {1..3}; do
    for j in {1..3}; do
        for k in {1..3}; do
            echo "$i-$j-$k"
            if [ $k -eq 2 ]; then
                break 2  # Break out of 2 levels
            fi
        done
    done
done

# break 1 = break (default)
# break 2 = break out of 2 loops
# break 3 = break out of 3 loops
```

---

## Nested Loops

### Basic Nested Loops

```bash
# Multiplication table
for i in {1..10}; do
    for j in {1..10}; do
        result=$((i * j))
        printf "%4d" $result
    done
    echo
done

# Process matrix
rows=(1 2 3)
cols=(A B C)

for row in "${rows[@]}"; do
    for col in "${cols[@]}"; do
        echo "Cell: $row$col"
    done
done
```

### File Processing

```bash
# Process files in multiple directories
for dir in dir1 dir2 dir3; do
    echo "Processing directory: $dir"
    for file in "$dir"/*.txt; do
        [ -f "$file" ] || continue
        echo "  File: $file"
        # Process file
    done
done

# Compare files
for file1 in set1/*.txt; do
    for file2 in set2/*.txt; do
        echo "Comparing: $(basename $file1) vs $(basename $file2)"
        diff "$file1" "$file2" > /dev/null || echo "  Different!"
    done
done
```

---

## Practical Examples

### Process Multiple Files

```bash
#!/bin/bash

# Batch rename files
for file in *.txt; do
    [ -f "$file" ] || continue
    newname="${file%.txt}_backup.txt"
    mv "$file" "$newname"
    echo "Renamed: $file -> $newname"
done

# Convert images
for img in *.jpg; do
    [ -f "$img" ] || continue
    output="${img%.jpg}.png"
    convert "$img" "$output"
    echo "Converted: $img -> $output"
done
```

### System Monitoring

```bash
#!/bin/bash

# Monitor CPU usage
while true; do
    cpu=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
    echo "$(date '+%H:%M:%S') - CPU: ${cpu}%"
    
    if (( $(echo "$cpu > 80" | bc -l) )); then
        echo "WARNING: High CPU usage!"
        # Send alert
    fi
    
    sleep 5
done

# Monitor disk space
while true; do
    for disk in / /home /var; do
        usage=$(df -h "$disk" | awk 'NR==2 {print $5}' | sed 's/%//')
        if [ "$usage" -gt 90 ]; then
            echo "ALERT: $disk is ${usage}% full"
        fi
    done
    sleep 300  # Check every 5 minutes
done
```

### Batch Processing

```bash
#!/bin/bash

# Process log files
for logfile in /var/log/*.log; do
    [ -f "$logfile" ] || continue
    
    echo "Processing: $logfile"
    
    # Count errors
    error_count=$(grep -c "ERROR" "$logfile")
    echo "  Errors: $error_count"
    
    # Extract unique IPs
    grep "Failed login" "$logfile" | \
        awk '{print $NF}' | \
        sort -u > "failed_ips_$(basename $logfile).txt"
done

# Backup databases
databases=("db1" "db2" "db3")
backup_dir="/backup/$(date +%Y%m%d)"
mkdir -p "$backup_dir"

for db in "${databases[@]}"; do
    echo "Backing up: $db"
    mysqldump "$db" | gzip > "$backup_dir/${db}.sql.gz"
    
    if [ $? -eq 0 ]; then
        echo "  Success"
    else
        echo "  Failed"
    fi
done
```

### Parallel Processing

```bash
#!/bin/bash

# Process files in parallel
for file in *.dat; do
    [ -f "$file" ] || continue
    (
        echo "Processing $file in background"
        process_file "$file"
    ) &
done

# Wait for all background jobs
wait
echo "All files processed"

# Limit concurrent jobs
max_jobs=4
count=0

for item in {1..100}; do
    (
        process "$item"
    ) &
    
    ((count++))
    if [ $count -ge $max_jobs ]; then
        wait -n  # Wait for any job to finish
        ((count--))
    fi
done

wait
echo "Complete"
```

### Progress Indicator

```bash
#!/bin/bash

# Progress bar
total=100
for i in $(seq 1 $total); do
    # Do work
    sleep 0.05
    
    # Show progress
    percent=$((i * 100 / total))
    filled=$((i * 50 / total))
    printf "\r["
    printf "%${filled}s" | tr ' ' '='
    printf "%$((50-filled))s" | tr ' ' ' '
    printf "] %d%%" "$percent"
done
echo ""
echo "Complete!"
```

---

## Summary

### Loop Syntax Comparison

```bash
# for loop - iterate over list
for item in list; do
    commands
done

# C-style for
for ((i=0; i<10; i++)); do
    commands
done

# while loop - while condition true
while [ condition ]; do
    commands
done

# until loop - until condition true
until [ condition ]; do
    commands
done
```

### Common Patterns

```bash
# Iterate files
for file in *.txt; do
    [ -f "$file" ] && process "$file"
done

# Read file lines
while read -r line; do
    echo "$line"
done < file.txt

# Counter loop
for i in {1..10}; do
    echo "$i"
done

# Infinite loop
while true; do
    commands
    sleep 1
done

# Wait for condition
until [ -f /tmp/ready ]; do
    sleep 1
done

# Break early
for i in {1..100}; do
    [ condition ] && break
done

# Skip iteration
for i in {1..10}; do
    [ condition ] && continue
    process "$i"
done
```

### Best Practices

1. **Always check file existence** in file loops
2. **Use `read -r`** when reading files
3. **Quote variables** in conditions
4. **Use `while read`** instead of `for` with command substitution for large outputs
5. **Add sleep** in infinite loops to avoid high CPU usage
6. **Use `break`** to exit loops early when condition met
7. **Use background jobs** (`&`) for parallel processing

---

**Next Chapter:** [Case Statements →](11-case-statements.md)

---

*Practice Exercise: Create a script that monitors a directory, processes new files as they appear, uses a menu system with loops, and implements a progress bar for long operations.*
