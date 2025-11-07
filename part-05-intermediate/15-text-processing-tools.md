# Chapter 15: Text Processing Tools

## Introduction

Text processing is one of Bash's most powerful capabilities. This chapter covers the essential command-line tools that make Bash indispensable for log analysis, data transformation, configuration management, and report generation. You'll master grep, sed, awk, and complementary utilities that form the backbone of text manipulation in Unix/Linux systems.

## Table of Contents
- [grep: Pattern Searching](#grep-pattern-searching)
- [sed: Stream Editor](#sed-stream-editor)
- [awk: Pattern Processing Language](#awk-pattern-processing-language)
- [cut: Column Extraction](#cut-column-extraction)
- [sort: Sorting Data](#sort-sorting-data)
- [uniq: Managing Duplicates](#uniq-managing-duplicates)
- [tr: Character Translation](#tr-character-translation)
- [Combining Tools](#combining-tools)
- [Real-World Applications](#real-world-applications)

---

## grep: Pattern Searching

### Basic grep Usage

```bash
# Search for pattern in file
grep "error" application.log

# Case-insensitive search
grep -i "error" application.log

# Invert match (show non-matching lines)
grep -v "debug" application.log

# Count matches
grep -c "error" application.log

# Show line numbers
grep -n "error" application.log

# Search multiple files
grep "error" *.log

# Recursive search
grep -r "TODO" /project/src/

# Show only filenames with matches
grep -l "error" *.log

# Show context (before and after)
grep -A 3 "error" log.txt    # 3 lines after
grep -B 2 "error" log.txt    # 2 lines before
grep -C 2 "error" log.txt    # 2 lines before and after
```

### Extended Regular Expressions

```bash
# Extended regex (-E flag)
grep -E "error|warning|critical" application.log

# Match at beginning of line
grep "^Error" log.txt

# Match at end of line
grep "failed$" log.txt

# Match word boundaries
grep -w "error" log.txt

# Match multiple patterns
grep -e "error" -e "warning" -e "critical" log.txt

# Exclude pattern
grep "error" log.txt | grep -v "ignored"

# Highlight matches
grep --color=auto "error" log.txt

# Match whole line
grep -x "exact line to match" file.txt
```

### Perl-Compatible Regular Expressions (PCRE)

```bash
# PCRE with -P flag
grep -P "\d{3}-\d{3}-\d{4}" contacts.txt    # Phone numbers
grep -P "^\d+$" numbers.txt                  # Only digits
grep -P "\b[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}\b" -i emails.txt

# Lookahead and lookbehind
grep -P "(?<=user: )\w+" log.txt            # After "user: "
grep -P "\d+(?= errors)" log.txt             # Before " errors"

# Named groups
grep -P "(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})" dates.txt
```

### Practical grep Examples

```bash
# Find all IPs in log file
grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" access.log

# Find all email addresses
grep -oE "\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b" file.txt

# Count unique error messages
grep "ERROR" app.log | sort | uniq -c | sort -nr

# Find files containing specific text
find . -type f -name "*.sh" -exec grep -l "#!/bin/bash" {} +

# Search compressed files
zgrep "error" logfile.gz

# Search with file type exclusion
grep -r "TODO" --exclude="*.min.js" --exclude-dir="node_modules" .
```

---

## sed: Stream Editor

### Basic Substitution

```bash
# Replace first occurrence per line
sed 's/old/new/' file.txt

# Replace all occurrences (global)
sed 's/old/new/g' file.txt

# Replace only on specific line
sed '5s/old/new/' file.txt

# Replace in range of lines
sed '10,20s/old/new/g' file.txt

# Case-insensitive replacement
sed 's/old/new/gi' file.txt

# In-place editing
sed -i 's/old/new/g' file.txt              # Linux
sed -i '' 's/old/new/g' file.txt           # macOS

# Create backup before editing
sed -i.bak 's/old/new/g' file.txt
```

### Advanced Substitution

```bash
# Use different delimiter
sed 's|/old/path|/new/path|g' file.txt
sed 's#http://#https://#g' urls.txt

# Use capture groups
sed 's/\([0-9]*\)-\([0-9]*\)/\2-\1/' file.txt

# Replace with special characters
sed 's/$/\r/' unix.txt                     # Add CR (DOS line endings)
sed 's/\t/    /g' file.txt                 # Replace tabs with spaces

# Multiple replacements
sed -e 's/foo/bar/g' -e 's/baz/qux/g' file.txt
sed 's/foo/bar/g; s/baz/qux/g' file.txt

# Reference matched pattern
sed 's/\(pattern\)/[\1]/g' file.txt        # Wrap pattern in brackets
```

### Deletion and Insertion

```bash
# Delete lines
sed '5d' file.txt                          # Delete line 5
sed '10,20d' file.txt                      # Delete lines 10-20
sed '/pattern/d' file.txt                  # Delete lines matching pattern
sed '/^$/d' file.txt                       # Delete empty lines
sed '/^#/d' file.txt                       # Delete comments

# Insert lines
sed '5i\New line before 5' file.txt        # Insert before line 5
sed '5a\New line after 5' file.txt         # Insert after line 5
sed '/pattern/i\New line' file.txt         # Insert before pattern
sed '/pattern/a\New line' file.txt         # Insert after pattern

# Change lines
sed '5c\Replacement line' file.txt         # Replace line 5
sed '/pattern/c\New content' file.txt      # Replace matching lines
```

### Printing and Displaying

```bash
# Print specific lines
sed -n '5p' file.txt                       # Print only line 5
sed -n '10,20p' file.txt                   # Print lines 10-20
sed -n '/pattern/p' file.txt               # Print matching lines

# Print line numbers
sed -n '=' file.txt                        # Print line numbers only
sed -n '/pattern/{=;p}' file.txt           # Print line number and content

# Show only changed lines
sed -n 's/old/new/gp' file.txt

# Multiple commands
sed -n '5p; 10p; 15p' file.txt
```

### Real-World sed Examples

```bash
# Remove HTML tags
sed 's/<[^>]*>//g' webpage.html

# Extract email addresses
sed -n 's/.*\([a-zA-Z0-9._%+-]*@[a-zA-Z0-9.-]*\.[a-zA-Z]*\).*/\1/p' file.txt

# Add line numbers
sed = file.txt | sed 'N; s/\n/\t/'

# Convert DOS to Unix line endings
sed 's/\r$//' dosfile.txt > unixfile.txt

# Comment out lines
sed 's/^/# /' file.txt                     # Add # to beginning
sed '/pattern/s/^/# /' file.txt            # Comment matching lines

# Uncomment lines
sed 's/^# //' file.txt

# Double-space file
sed 'G' file.txt

# Remove duplicate consecutive lines
sed '$!N; /^\(.*\)\n\1$/!P; D'

# Trim whitespace
sed 's/^[[:space:]]*//; s/[[:space:]]*$//' file.txt
```

---

## awk: Pattern Processing Language

### Basic awk Syntax

```bash
# Print specific fields
awk '{print $1}' file.txt                  # First field
awk '{print $1, $3}' file.txt              # First and third fields
awk '{print $NF}' file.txt                 # Last field
awk '{print $(NF-1)}' file.txt             # Second-to-last field

# Field separator
awk -F: '{print $1}' /etc/passwd           # Use : as separator
awk -F',' '{print $2}' data.csv            # Use comma for CSV

# Output separator
awk '{print $1, $2}' file.txt              # Default space separator
awk -v OFS=',' '{print $1, $2}' file.txt   # Comma separator
```

### Pattern Matching

```bash
# Match pattern
awk '/error/ {print}' log.txt
awk '/error/' log.txt                      # Shorthand

# Negate pattern
awk '!/debug/' log.txt

# Multiple patterns
awk '/error/ || /warning/' log.txt

# Pattern with action
awk '/error/ {print $1, $5}' log.txt

# Range patterns
awk '/START/,/END/' file.txt               # From START to END

# Field matching
awk '$3 == "active"' status.txt
awk '$2 > 100' numbers.txt
awk '$1 ~ /^[A-Z]/' names.txt              # Regex match
awk '$1 !~ /test/' file.txt                # Regex not match
```

### Variables and Operators

```bash
# Built-in variables
awk '{print NR, $0}' file.txt              # NR = line number
awk '{print NF}' file.txt                  # NF = number of fields
awk 'END {print NR}' file.txt              # Total lines
awk '{print FILENAME, $0}' file.txt        # FILENAME

# User variables
awk '{sum += $1} END {print sum}' numbers.txt
awk '{count++} END {print count}' file.txt

# Arithmetic
awk '{print $1 + $2}' numbers.txt
awk '{print $1 * $2}' numbers.txt
awk '{avg = ($1 + $2 + $3) / 3; print avg}' scores.txt

# Conditionals
awk '{if ($3 > 100) print $1, $3}' data.txt
awk '{print ($1 > 50) ? "high" : "low"}' numbers.txt
```

### BEGIN and END Blocks

```bash
# BEGIN block (before processing)
awk 'BEGIN {print "Name\tScore"} {print $1, $2}' scores.txt

# END block (after processing)
awk '{sum += $1} END {print "Total:", sum}' numbers.txt

# Both BEGIN and END
awk 'BEGIN {print "Processing..."} {count++} END {print "Total:", count}' file.txt

# Set variables in BEGIN
awk 'BEGIN {FS=":"; OFS="\t"} {print $1, $7}' /etc/passwd
```

### Advanced awk Features

```bash
# Arrays
awk '{count[$1]++} END {for (word in count) print word, count[word]}' words.txt

# String functions
awk '{print length($0)}' file.txt          # String length
awk '{print toupper($1)}' file.txt         # Uppercase
awk '{print tolower($1)}' file.txt         # Lowercase
awk '{print substr($1, 1, 3)}' file.txt    # Substring

# Math functions
awk '{print sqrt($1)}' numbers.txt
awk '{print int($1)}' decimals.txt

# Formatting
awk '{printf "%-10s %5d\n", $1, $2}' data.txt

# Multiple files
awk 'FNR==1{print "Processing", FILENAME} {print}' file1.txt file2.txt
```

### Real-World awk Examples

```bash
# Calculate average
awk '{sum += $1; count++} END {print sum/count}' numbers.txt

# Sum column
awk '{sum += $3} END {print "Total:", sum}' sales.csv

# Count occurrences
awk '{count[$1]++} END {for (i in count) print i, count[i]}' data.txt

# Top 10 IPs from access log
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -10

# Extract specific columns from CSV
awk -F',' '{print $2, $5, $7}' data.csv

# Calculate percentage
awk '{pct = ($2 / $3) * 100; printf "%s: %.2f%%\n", $1, pct}' data.txt

# Group by field and sum
awk '{sales[$1] += $2} END {for (i in sales) print i, sales[i]}' sales.txt

# Print every Nth line
awk 'NR % 5 == 0' file.txt                 # Every 5th line

# Process log file
awk '/ERROR/ {errors++} /WARNING/ {warnings++} END {
    print "Errors:", errors
    print "Warnings:", warnings
}' application.log
```

---

## cut: Column Extraction

```bash
# Extract by character position
cut -c1-10 file.txt                        # Characters 1-10
cut -c1,5,10 file.txt                      # Characters 1, 5, and 10
cut -c5- file.txt                          # From character 5 to end

# Extract by field (default delimiter: tab)
cut -f1 file.txt                           # First field
cut -f1,3 file.txt                         # Fields 1 and 3
cut -f1-3 file.txt                         # Fields 1 through 3

# Custom delimiter
cut -d':' -f1 /etc/passwd                  # Use : as delimiter
cut -d',' -f2,4 data.csv                   # CSV with comma

# Output delimiter
cut -d':' -f1,7 --output-delimiter=$'\t' /etc/passwd

# Complement (everything except)
cut -d',' -f1 --complement data.csv        # All except field 1

# Examples
cut -d' ' -f1 access.log                   # Extract IPs
cut -d':' -f1,3 /etc/passwd                # Username and UID
echo "one,two,three" | cut -d',' -f2       # Extract "two"
```

---

## sort: Sorting Data

```bash
# Basic sort
sort file.txt                              # Alphabetical
sort -r file.txt                           # Reverse order

# Numeric sort
sort -n numbers.txt                        # Numeric order
sort -n -r numbers.txt                     # Reverse numeric

# Sort by field
sort -k2 file.txt                          # Sort by 2nd field
sort -k2,2 file.txt                        # Only 2nd field
sort -t: -k3n /etc/passwd                  # By UID (numeric)

# Unique sort
sort -u file.txt                           # Remove duplicates

# Case-insensitive
sort -f file.txt

# Month sort
sort -M months.txt                         # Jan, Feb, Mar order

# Human-readable numbers
sort -h sizes.txt                          # 1K, 2M, 3G

# Check if sorted
sort -c file.txt                           # Check if already sorted

# Stable sort
sort -s file.txt                           # Maintain original order for equal

# Examples
sort -t',' -k2nr sales.csv                 # Sort CSV by 2nd field, numeric, reverse
sort -t: -k3n /etc/passwd | head -5        # First 5 users by UID
ps aux | sort -k3nr | head -10             # Top CPU processes
du -h | sort -hr | head -10                # Largest directories
```

---

## uniq: Managing Duplicates

```bash
# Remove consecutive duplicates
uniq file.txt

# Count occurrences
uniq -c file.txt

# Show only duplicates
uniq -d file.txt

# Show only unique lines
uniq -u file.txt

# Ignore case
uniq -i file.txt

# Skip fields
uniq -f1 file.txt                          # Skip first field

# Check specific characters
uniq -w5 file.txt                          # Compare only first 5 chars

# Examples (usually with sort)
sort file.txt | uniq                       # Remove all duplicates
sort file.txt | uniq -c                    # Count occurrences
sort file.txt | uniq -c | sort -nr         # Most common first
sort file.txt | uniq -d                    # Show only duplicates
cat access.log | cut -d' ' -f1 | sort | uniq -c | sort -nr | head -10
```

---

## tr: Character Translation

```bash
# Character replacement
echo "hello" | tr 'a-z' 'A-Z'              # Lowercase to uppercase
echo "HELLO" | tr 'A-Z' 'a-z'              # Uppercase to lowercase
echo "hello world" | tr ' ' '_'            # Replace spaces

# Delete characters
echo "hello123world" | tr -d '0-9'         # Delete digits
echo "hello world" | tr -d ' '             # Delete spaces

# Squeeze repeats
echo "hello    world" | tr -s ' '          # Squeeze multiple spaces
echo "hellooo" | tr -s 'o'                 # Squeeze repeated 'o'

# Complement (everything except)
echo "hello123" | tr -cd '0-9'             # Keep only digits
echo "hello world" | tr -cd 'a-zA-Z '      # Keep only letters and spaces

# Character classes
echo "Hello123" | tr '[:lower:]' '[:upper:]'
echo "Hello World" | tr '[:upper:]' '[:lower:]'
echo "hello world" | tr '[:alpha:]' '*'

# Examples
tr '\n' ' ' < file.txt                     # Join lines
tr -s '\n' < file.txt                      # Remove blank lines
echo "PATH" | tr ':' '\n'                  # Split PATH
cat file.txt | tr -d '\r'                  # DOS to Unix
```

---

## Combining Tools

### Powerful Pipelines

```bash
# Top 10 most frequent words
cat file.txt | tr -s ' ' '\n' | sort | uniq -c | sort -nr | head -10

# Extract and count IP addresses
grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" access.log | sort | uniq -c | sort -nr

# Find largest files
find . -type f -exec du -h {} + | sort -hr | head -20

# Process CSV: extract, filter, sort
cut -d',' -f1,3 data.csv | grep "active" | sort -t',' -k2

# Analyze log file
cat app.log | grep ERROR | awk '{print $5}' | sort | uniq -c | sort -nr

# Count file extensions
find . -type f | sed 's/.*\.//' | sort | uniq -c | sort -nr

# Monitor changing file
tail -f /var/log/syslog | grep --line-buffered "error" | awk '{print $1, $2, $5}'
```

---

## Real-World Applications

### Log Analysis

```bash
#!/bin/bash

# Apache access log analyzer
analyze_apache_log() {
    local log_file="$1"
    
    echo "=== Apache Log Analysis ==="
    echo
    
    echo "Top 10 IP Addresses:"
    awk '{print $1}' "$log_file" | sort | uniq -c | sort -nr | head -10
    echo
    
    echo "Top 10 Requested URLs:"
    awk '{print $7}' "$log_file" | sort | uniq -c | sort -nr | head -10
    echo
    
    echo "HTTP Status Codes:"
    awk '{print $9}' "$log_file" | sort | uniq -c | sort -nr
    echo
    
    echo "Requests per Hour:"
    awk '{print $4}' "$log_file" | cut -d: -f2 | sort | uniq -c
    echo
    
    echo "User Agents:"
    awk -F'"' '{print $6}' "$log_file" | sort | uniq -c | sort -nr | head -5
}

# Error log analyzer
analyze_error_log() {
    local log_file="$1"
    
    echo "=== Error Analysis ==="
    echo "Total Errors: $(grep -c "ERROR" "$log_file")"
    echo "Total Warnings: $(grep -c "WARN" "$log_file")"
    echo
    
    echo "Error Distribution:"
    grep "ERROR" "$log_file" | awk '{print $5}' | sort | uniq -c | sort -nr
    echo
    
    echo "Errors Timeline:"
    grep "ERROR" "$log_file" | awk '{print $1, $2}' | cut -d: -f1 | sort | uniq -c
}
```

### CSV Processing

```bash
#!/bin/bash

# Process sales data
process_sales_csv() {
    local csv_file="$1"
    
    # Skip header, calculate totals
    awk -F',' 'NR>1 {
        sales[$2] += $4
        count[$2]++
    } END {
        print "Product,Total Sales,Count,Average"
        for (product in sales) {
            avg = sales[product] / count[product]
            printf "%s,%.2f,%d,%.2f\n", product, sales[product], count[product], avg
        }
    }' "$csv_file" | sort -t',' -k2 -nr
}

# Clean and validate CSV
clean_csv() {
    local input="$1"
    local output="$2"
    
    # Remove empty lines, trim whitespace, fix quotes
    sed '/^$/d' "$input" | \
    awk '{gsub(/^[ \t]+|[ \t]+$/, ""); print}' | \
    sed 's/"\([^"]*\)"/\1/g' > "$output"
}

# Extract columns
extract_csv_columns() {
    local file="$1"
    shift
    local columns="$@"
    
    awk -F',' -v cols="$columns" '
    BEGIN {
        split(cols, arr, " ")
    }
    {
        for (i in arr) {
            printf "%s%s", $arr[i], (i < length(arr) ? "," : "\n")
        }
    }' "$file"
}
```

### Configuration File Updates

```bash
#!/bin/bash

# Update configuration value
update_config() {
    local file="$1"
    local key="$2"
    local value="$3"
    
    if grep -q "^${key}=" "$file"; then
        # Update existing
        sed -i "s/^${key}=.*/${key}=${value}/" "$file"
    else
        # Add new
        echo "${key}=${value}" >> "$file"
    fi
}

# Comment out configuration
comment_config() {
    local file="$1"
    local pattern="$2"
    
    sed -i "/^${pattern}/s/^/# /" "$file"
}

# Uncomment configuration
uncomment_config() {
    local file="$1"
    local pattern="$2"
    
    sed -i "/^# ${pattern}/s/^# //" "$file"
}

# Extract value from config
get_config_value() {
    local file="$1"
    local key="$2"
    
    grep "^${key}=" "$file" | cut -d'=' -f2
}
```

### Report Generation

```bash
#!/bin/bash

# Generate system report
generate_system_report() {
    local output="system_report_$(date +%Y%m%d).txt"
    
    {
        echo "=== System Report ==="
        echo "Generated: $(date)"
        echo
        
        echo "=== Disk Usage ==="
        df -h | awk 'NR==1 || $5+0 > 80'
        echo
        
        echo "=== Memory Usage ==="
        free -h | awk 'NR==1 || NR==2'
        echo
        
        echo "=== Top Processes by CPU ==="
        ps aux | sort -k3 -nr | head -10 | awk '{printf "%-10s %5s %5s %s\n", $1, $3"%", $4"%", $11}'
        echo
        
        echo "=== Top Processes by Memory ==="
        ps aux | sort -k4 -nr | head -10 | awk '{printf "%-10s %5s %5s %s\n", $1, $3"%", $4"%", $11}'
        echo
        
        echo "=== Failed Login Attempts ==="
        grep "Failed password" /var/log/auth.log | awk '{print $1, $2, $11}' | sort | uniq -c | sort -nr | head -10
        
    } > "$output"
    
    echo "Report saved to: $output"
}
```

---

## Summary

In this chapter, you learned:

- ✅ **grep**: Powerful pattern searching with regex support
- ✅ **sed**: Stream editing for text transformation
- ✅ **awk**: Complete text processing programming language
- ✅ **cut**: Simple column extraction
- ✅ **sort**: Versatile sorting with multiple options
- ✅ **uniq**: Duplicate management
- ✅ **tr**: Character translation and deletion
- ✅ **Pipelines**: Combining tools for complex processing
- ✅ **Real-World**: Log analysis, CSV processing, reporting

### Key Takeaways

| Tool | Best For | Common Use |
|------|----------|------------|
| grep | Finding patterns | Log analysis, searching |
| sed | Text replacement | Config updates, formatting |
| awk | Columnar data | Reports, calculations |
| cut | Simple extraction | CSV fields, log parsing |
| sort | Ordering data | Rankings, deduplication |
| uniq | Duplicates | Counting, filtering |
| tr | Character ops | Case conversion, cleanup |

### Next Steps

In [Chapter 16: Working with Arrays](./16-working-with-arrays.md), you'll learn how to use Bash arrays for managing collections of data efficiently.

---

## Practice Exercises

1. **Exercise 1**: Extract all unique IP addresses from an access log and count occurrences
2. **Exercise 2**: Parse a CSV file and calculate the sum of a specific column
3. **Exercise 3**: Find the top 10 most common words in a text file
4. **Exercise 4**: Create a script that updates configuration values in an INI file
5. **Exercise 5**: Generate a report of the largest files in a directory tree
