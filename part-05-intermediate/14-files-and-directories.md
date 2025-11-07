# Chapter 14: Working with Files and Directories

## Introduction

File and directory operations are fundamental to system administration and automation. This chapter covers advanced file testing operators, manipulation techniques, and practical automation scenarios for managing file systems efficiently.

## Table of Contents
- [File Testing Operators](#file-testing-operators)
- [Advanced File Operations](#advanced-file-operations)
- [File Descriptors and Redirection](#file-descriptors-and-redirection)
- [Directory Operations](#directory-operations)
- [Links and References](#links-and-references)
- [File Automation](#file-automation)
- [Practical Examples](#practical-examples)

---

## File Testing Operators

### Complete Test Operators Reference

| Operator | Description | Example |
|----------|-------------|---------|
| `-e` | File exists | `[ -e file.txt ]` |
| `-f` | Regular file exists | `[ -f file.txt ]` |
| `-d` | Directory exists | `[ -d /path/dir ]` |
| `-L` | Symbolic link exists | `[ -L link ]` |
| `-r` | File is readable | `[ -r file.txt ]` |
| `-w` | File is writable | `[ -w file.txt ]` |
| `-x` | File is executable | `[ -x script.sh ]` |
| `-s` | File exists and not empty | `[ -s file.txt ]` |
| `-N` | Modified since last read | `[ -N file.txt ]` |
| `-O` | Owned by current user | `[ -O file.txt ]` |
| `-G` | Group matches current | `[ -G file.txt ]` |
| `-nt` | File1 newer than file2 | `[ file1 -nt file2 ]` |
| `-ot` | File1 older than file2 | `[ file1 -ot file2 ]` |
| `-ef` | Files have same inode | `[ file1 -ef file2 ]` |

### Basic File Tests

```bash
#!/bin/bash

file="data.txt"

# Check if file exists
if [ -e "$file" ]; then
    echo "✓ File exists: $file"
else
    echo "✗ File not found: $file"
fi

# Check if regular file
if [ -f "$file" ]; then
    echo "✓ Regular file"
fi

# Check if directory
if [ -d "/tmp" ]; then
    echo "✓ /tmp is a directory"
fi

# Check if readable
if [ -r "$file" ]; then
    echo "✓ File is readable"
fi

# Check if writable
if [ -w "$file" ]; then
    echo "✓ File is writable"
fi

# Check if executable
if [ -x "script.sh" ]; then
    echo "✓ Script is executable"
fi

# Check if empty
if [ -s "$file" ]; then
    echo "✓ File has content"
else
    echo "✗ File is empty"
fi
```

### Combined Tests

```bash
#!/bin/bash

check_file() {
    local file="$1"
    
    # Multiple conditions
    if [ -f "$file" ] && [ -r "$file" ] && [ -s "$file" ]; then
        echo "✓ $file: exists, readable, and has content"
        return 0
    else
        echo "✗ $file: failed one or more checks"
        return 1
    fi
}

# Ownership tests
check_ownership() {
    local file="$1"
    
    if [ -O "$file" ]; then
        echo "✓ You own: $file"
    fi
    
    if [ -G "$file" ]; then
        echo "✓ Your group matches: $file"
    fi
}

# Age comparison
compare_files() {
    local file1="$1"
    local file2="$2"
    
    if [ "$file1" -nt "$file2" ]; then
        echo "$file1 is newer than $file2"
    elif [ "$file1" -ot "$file2" ]; then
        echo "$file1 is older than $file2"
    else
        echo "Files have same modification time"
    fi
}

# Usage
check_file "config.txt"
check_ownership "myfile.txt"
compare_files "version1.txt" "version2.txt"
```

---

## Advanced File Operations

### Finding Files

```bash
#!/bin/bash

# Find by name
find /path/to/search -name "*.txt"

# Find by type
find /path -type f    # Files only
find /path -type d    # Directories only
find /path -type l    # Symbolic links only

# Find by size
find /path -size +100M    # Larger than 100MB
find /path -size -1K      # Smaller than 1KB
find /path -size 50M      # Exactly 50MB

# Find by modification time
find /path -mtime -7      # Modified in last 7 days
find /path -mtime +30     # Modified more than 30 days ago
find /path -mmin -60      # Modified in last 60 minutes

# Find by permissions
find /path -perm 644      # Exact permissions
find /path -perm -644     # At least these permissions
find /path -perm /644     # Any of these permissions

# Find and execute
find /path -name "*.log" -exec rm {} \;
find /path -name "*.txt" -exec grep "error" {} +

# Find with multiple conditions
find /path -type f -name "*.sh" -size +1M -mtime -30
```

### Advanced Find Examples

```bash
#!/bin/bash

# Find and list with details
find_with_details() {
    local dir="$1"
    find "$dir" -type f -printf "%p\t%s bytes\t%Tc\n" | column -t
}

# Find large files
find_large_files() {
    local size="${1:-100M}"
    echo "Finding files larger than $size..."
    find . -type f -size +"$size" -exec ls -lh {} \; | \
        awk '{ print $9 ": " $5 }'
}

# Find duplicate files by size and hash
find_duplicates() {
    find . -type f -exec md5sum {} + | \
        sort | \
        uniq -w32 -D --all-repeated=separate
}

# Find empty files and directories
find_empty() {
    echo "Empty files:"
    find . -type f -empty
    
    echo -e "\nEmpty directories:"
    find . -type d -empty
}

# Find files modified today
find_today() {
    find . -type f -mtime 0
}

# Find and organize by extension
organize_by_extension() {
    for ext in txt pdf jpg png; do
        echo ".$ext files:"
        find . -type f -name "*.$ext" | wc -l
    done
}
```

### Reading Files

```bash
#!/bin/bash

# Read line by line
while IFS= read -r line; do
    echo "Line: $line"
done < file.txt

# Read into array
mapfile -t lines < file.txt
# Or: readarray -t lines < file.txt

# Read with line numbers
while IFS= read -r line; do
    echo "$((++count)): $line"
done < file.txt

# Read CSV file
while IFS=, read -r col1 col2 col3; do
    echo "Column 1: $col1, Column 2: $col2, Column 3: $col3"
done < data.csv

# Skip header line
{
    read  # Skip first line
    while IFS=, read -r name age city; do
        echo "$name is $age years old from $city"
    done
} < users.csv

# Read last N lines
tail -n 10 file.txt | while read line; do
    echo "$line"
done
```

### Writing and Modifying Files

```bash
#!/bin/bash

# Write to file (overwrite)
echo "Content" > file.txt

# Append to file
echo "More content" >> file.txt

# Write multiple lines
cat > file.txt << 'EOF'
Line 1
Line 2
Line 3
EOF

# Write with variables expanded
cat > config.txt << EOF
User: $USER
Home: $HOME
Date: $(date)
EOF

# Write array to file
declare -a items=("apple" "banana" "cherry")
printf "%s\n" "${items[@]}" > fruits.txt

# In-place file modification
sed -i 's/old/new/g' file.txt           # Linux
sed -i '' 's/old/new/g' file.txt        # macOS

# Add line to beginning of file
sed -i '1i\New first line' file.txt

# Add line to end of file
echo "Last line" >> file.txt

# Delete lines matching pattern
sed -i '/pattern/d' file.txt

# Replace line number
sed -i '5s/.*/New line 5/' file.txt
```

---

## File Descriptors and Redirection

### Understanding File Descriptors

| FD | Name | Description |
|----|------|-------------|
| 0 | stdin | Standard input |
| 1 | stdout | Standard output |
| 2 | stderr | Standard error |
| 3-9 | custom | User-defined |

### Advanced Redirection

```bash
#!/bin/bash

# Redirect stdout
command > output.txt

# Redirect stderr
command 2> error.txt

# Redirect both stdout and stderr
command > output.txt 2>&1
command &> output.txt  # Shorter syntax

# Redirect to different files
command > stdout.txt 2> stderr.txt

# Append
command >> output.txt 2>> error.txt

# Redirect stdout to stderr
echo "Error message" >&2

# Redirect stderr to stdout
command 2>&1 | grep "pattern"

# Discard output
command > /dev/null
command 2> /dev/null
command &> /dev/null

# Here document
cat << EOF > file.txt
Content here
EOF

# Here string
grep "pattern" <<< "$variable"
```

### Custom File Descriptors

```bash
#!/bin/bash

# Open file for reading (FD 3)
exec 3< input.txt
while read -u 3 line; do
    echo "$line"
done
exec 3<&-  # Close FD 3

# Open file for writing (FD 4)
exec 4> output.txt
echo "Line 1" >&4
echo "Line 2" >&4
exec 4>&-  # Close FD 4

# Open for read/write (FD 5)
exec 5<> datafile.txt
read -u 5 line
echo "New line" >&5
exec 5>&-

# Swap stdout and stderr
exec 3>&1 1>&2 2>&3 3>&-

# Save and restore stdout
exec 6>&1       # Save stdout to FD 6
exec > log.txt  # Redirect stdout to file
echo "This goes to log.txt"
exec 1>&6       # Restore stdout
exec 6>&-       # Close FD 6
echo "This goes to terminal"
```

---

## Directory Operations

### Creating Directories

```bash
#!/bin/bash

# Create single directory
mkdir new_dir

# Create nested directories
mkdir -p path/to/nested/dir

# Create with specific permissions
mkdir -m 755 new_dir

# Create multiple directories
mkdir dir1 dir2 dir3

# Create directory structure
mkdir -p project/{src,lib,bin,docs,tests}
mkdir -p project/src/{main,utils,models}

# Create with date stamp
mkdir "backup_$(date +%Y%m%d)"

# Create and cd into it
mkdir new_dir && cd new_dir

# Create temporary directory
temp_dir=$(mktemp -d)
echo "Created temp dir: $temp_dir"
```

### Directory Navigation

```bash
#!/bin/bash

# Save current directory
pushd /path/to/dir
# Do work...
popd  # Return to previous directory

# Change to previous directory
cd -

# Go to home directory
cd ~
cd

# Go to user's home
cd ~username

# Directory stack
pushd /tmp
pushd /var
pushd /etc
dirs -v  # Show directory stack
popd     # Return to /var
popd     # Return to /tmp
popd     # Return to original
```

### Directory Analysis

```bash
#!/bin/bash

# Count files in directory
count_files() {
    local dir="${1:-.}"
    find "$dir" -type f | wc -l
}

# Calculate directory size
dir_size() {
    local dir="${1:-.}"
    du -sh "$dir" 2>/dev/null | cut -f1
}

# List largest directories
largest_dirs() {
    local count="${1:-10}"
    du -h --max-depth=1 | sort -hr | head -n "$count"
}

# Directory tree
tree_view() {
    local dir="${1:-.}"
    local prefix="${2:-}"
    
    for item in "$dir"/*; do
        [ -e "$item" ] || continue
        basename "$item"
        if [ -d "$item" ]; then
            tree_view "$item" "  $prefix"
        fi
    done
}

# Find empty directories
find_empty_dirs() {
    find "${1:-.}" -type d -empty
}

# Count files by extension
count_by_extension() {
    find . -type f | sed 's/.*\.//' | sort | uniq -c | sort -nr
}
```

---

## Links and References

### Symbolic Links

```bash
#!/bin/bash

# Create symbolic link
ln -s /path/to/original /path/to/link

# Create symbolic link in current directory
ln -s /path/to/file .

# Force creation (overwrite if exists)
ln -sf /path/to/original /path/to/link

# Check if symbolic link
if [ -L "mylink" ]; then
    echo "It's a symbolic link"
fi

# Read link target
readlink mylink
readlink -f mylink  # Canonical path

# Find all symbolic links
find /path -type l

# Find broken symbolic links
find /path -type l ! -exec test -e {} \; -print

# Create relative symbolic link
ln -s ../config/app.conf config_link
```

### Hard Links

```bash
#!/bin/bash

# Create hard link
ln /path/to/original /path/to/hardlink

# Check if files are hard linked (same inode)
if [ file1 -ef file2 ]; then
    echo "Files are hard linked"
fi

# Count hard links
stat -c %h filename

# Find all hard links to a file
inode=$(stat -c %i filename)
find /path -inum "$inode"
```

---

## File Automation

### Automated Log Rotation

```bash
#!/bin/bash

rotate_log() {
    local log_file="$1"
    local max_size="${2:-10485760}"  # 10MB default
    local keep_count="${3:-5}"
    
    # Check if rotation needed
    if [ ! -f "$log_file" ]; then
        touch "$log_file"
        return 0
    fi
    
    local size=$(stat -f%z "$log_file" 2>/dev/null || stat -c%s "$log_file")
    
    if [ "$size" -lt "$max_size" ]; then
        return 0
    fi
    
    echo "Rotating $log_file (size: $size bytes)"
    
    # Rotate existing logs
    for i in $(seq $((keep_count-1)) -1 1); do
        if [ -f "${log_file}.$i" ]; then
            mv "${log_file}.$i" "${log_file}.$((i+1))"
        fi
    done
    
    # Move current log
    mv "$log_file" "${log_file}.1"
    
    # Compress old logs
    gzip "${log_file}.1"
    
    # Delete oldest
    [ -f "${log_file}.$((keep_count+1))" ] && rm "${log_file}.$((keep_count+1))"
    
    # Create new log file
    touch "$log_file"
    
    echo "✓ Log rotation complete"
}

# Usage
rotate_log "/var/log/app.log" 10485760 5
```

### Automated Backup

```bash
#!/bin/bash

backup_directory() {
    local source="$1"
    local dest_base="${2:-/backup}"
    local timestamp=$(date +%Y%m%d_%H%M%S)
    local backup_name="backup_${timestamp}.tar.gz"
    local dest="$dest_base/$backup_name"
    
    # Validate source
    if [ ! -d "$source" ]; then
        echo "Error: Source directory not found: $source" >&2
        return 1
    fi
    
    # Create destination directory
    mkdir -p "$dest_base"
    
    # Create backup
    echo "Creating backup: $source -> $dest"
    
    if tar -czf "$dest" -C "$(dirname "$source")" "$(basename "$source")" 2>/dev/null; then
        local size=$(du -h "$dest" | cut -f1)
        echo "✓ Backup created: $dest ($size)"
        
        # Create checksum
        md5sum "$dest" > "${dest}.md5"
        
        # Cleanup old backups (keep last 7 days)
        find "$dest_base" -name "backup_*.tar.gz" -mtime +7 -delete
        find "$dest_base" -name "backup_*.md5" -mtime +7 -delete
        
        return 0
    else
        echo "✗ Backup failed" >&2
        return 1
    fi
}

# Incremental backup
incremental_backup() {
    local source="$1"
    local dest="$2"
    local snapshot_file="$dest/.snapshot"
    
    mkdir -p "$dest"
    
    rsync -av --delete \
        --link-dest="$dest/latest" \
        "$source/" \
        "$dest/$(date +%Y%m%d_%H%M%S)/"
    
    ln -snf "$(date +%Y%m%d_%H%M%S)" "$dest/latest"
}

# Usage
backup_directory "/home/user/documents" "/backup/docs"
incremental_backup "/data" "/backup/incremental"
```

### File Organization

```bash
#!/bin/bash

organize_downloads() {
    local download_dir="${1:-$HOME/Downloads}"
    
    # Create category directories
    mkdir -p "$download_dir"/{images,documents,videos,audio,archives,code}
    
    # Organize by extension
    find "$download_dir" -maxdepth 1 -type f | while read file; do
        case "${file,,}" in
            *.jpg|*.jpeg|*.png|*.gif|*.bmp|*.svg)
                mv "$file" "$download_dir/images/"
                ;;
            *.pdf|*.doc|*.docx|*.txt|*.odt)
                mv "$file" "$download_dir/documents/"
                ;;
            *.mp4|*.avi|*.mkv|*.mov|*.wmv)
                mv "$file" "$download_dir/videos/"
                ;;
            *.mp3|*.wav|*.flac|*.aac|*.ogg)
                mv "$file" "$download_dir/audio/"
                ;;
            *.zip|*.tar|*.gz|*.rar|*.7z)
                mv "$file" "$download_dir/archives/"
                ;;
            *.sh|*.py|*.js|*.java|*.c|*.cpp)
                mv "$file" "$download_dir/code/"
                ;;
        esac
    done
    
    echo "✓ Downloads organized"
}

# Organize by date
organize_by_date() {
    local source_dir="$1"
    
    find "$source_dir" -type f | while read file; do
        local date_dir=$(date -r "$file" +%Y/%m 2>/dev/null || \
                        stat -c %y "$file" | cut -d' ' -f1 | sed 's/-/\//g' | cut -d/ -f1-2)
        
        mkdir -p "$source_dir/$date_dir"
        mv "$file" "$source_dir/$date_dir/"
    done
}

# Bulk rename
bulk_rename() {
    local pattern="$1"
    local replacement="$2"
    local dir="${3:-.}"
    
    find "$dir" -name "$pattern" -type f | while read file; do
        local dir=$(dirname "$file")
        local name=$(basename "$file")
        local new_name="${name//$pattern/$replacement}"
        
        mv "$file" "$dir/$new_name"
        echo "Renamed: $name -> $new_name"
    done
}
```

---

## Practical Examples

### Disk Usage Analyzer

```bash
#!/bin/bash

disk_usage_report() {
    local dir="${1:-.}"
    local top_n="${2:-20}"
    
    echo "=== Disk Usage Report for $dir ==="
    echo "Generated: $(date)"
    echo
    
    echo "Top $top_n largest files:"
    find "$dir" -type f -exec du -h {} + | \
        sort -hr | \
        head -n "$top_n" | \
        nl
    
    echo
    echo "Top $top_n largest directories:"
    du -h "$dir"/* 2>/dev/null | \
        sort -hr | \
        head -n "$top_n" | \
        nl
    
    echo
    echo "Files by extension:"
    find "$dir" -type f | \
        sed 's/.*\.//' | \
        sort | \
        uniq -c | \
        sort -nr | \
        head -n 10
    
    echo
    echo "Total size: $(du -sh "$dir" | cut -f1)"
}

# Usage
disk_usage_report "/var/log" 10 > disk_report.txt
```

### Duplicate File Finder

```bash
#!/bin/bash

find_duplicates() {
    local dir="${1:-.}"
    
    echo "Searching for duplicates in: $dir"
    
    declare -A files_by_hash
    
    find "$dir" -type f -print0 | while IFS= read -r -d '' file; do
        hash=$(md5sum "$file" | cut -d' ' -f1)
        size=$(stat -c%s "$file")
        key="${size}_${hash}"
        
        if [ -n "${files_by_hash[$key]}" ]; then
            echo "Duplicate found:"
            echo "  Original: ${files_by_hash[$key]}"
            echo "  Duplicate: $file"
            echo
        else
            files_by_hash[$key]="$file"
        fi
    done
}
```

### File Integrity Checker

```bash
#!/bin/bash

create_checksums() {
    local dir="$1"
    local checksum_file="$dir/checksums.md5"
    
    echo "Creating checksums for: $dir"
    
    find "$dir" -type f ! -name "checksums.md5" -exec md5sum {} + > "$checksum_file"
    
    echo "✓ Checksums saved to: $checksum_file"
}

verify_checksums() {
    local dir="$1"
    local checksum_file="$dir/checksums.md5"
    
    if [ ! -f "$checksum_file" ]; then
        echo "Error: Checksum file not found" >&2
        return 1
    fi
    
    echo "Verifying file integrity..."
    
    md5sum -c "$checksum_file" 2>&1 | while read line; do
        if [[ "$line" == *"FAILED"* ]]; then
            echo "✗ $line"
        elif [[ "$line" == *"OK"* ]]; then
            echo "✓ $line"
        fi
    done
}

# Usage
create_checksums "/important/data"
verify_checksums "/important/data"
```

---

## Summary

In this chapter, you learned:

- ✅ **File Testing**: Complete set of test operators for file validation
- ✅ **Advanced Find**: Complex file searching and filtering
- ✅ **File I/O**: Reading, writing, and modifying files programmatically
- ✅ **File Descriptors**: Advanced redirection and custom FDs
- ✅ **Directory Ops**: Creating, analyzing, and managing directories
- ✅ **Links**: Symbolic and hard link management
- ✅ **Automation**: Log rotation, backups, and file organization

### Next Steps

In [Chapter 15: Text Processing Tools](./15-text-processing-tools.md), you'll master grep, sed, awk, and other powerful text manipulation utilities.

---

## Practice Exercises

1. Create a script that finds all files larger than 100MB modified in the last 7 days
2. Write a log rotation system that keeps the last 10 versions
3. Build a duplicate file finder that reports by size and checksum
4. Create an automated backup script with compression and verification
5. Develop a file organization tool that categorizes by type and date
