# Chapter 4: Basic Shell Commands

## Table of Contents
1. [Navigation Commands](#navigation-commands)
2. [File Operations](#file-operations)
3. [File Permissions](#file-permissions)
4. [Process Management](#process-management)
5. [Input/Output Redirection](#inputoutput-redirection)
6. [Combining Commands](#combining-commands)
7. [Summary](#summary)

---

## Navigation Commands

### pwd - Print Working Directory

```bash
# Show current directory
pwd
# Output: /home/user/documents

# Show physical path (resolve symlinks)
pwd -P

# Show logical path (with symlinks)
pwd -L
```

### cd - Change Directory

```bash
# Go to home directory
cd
cd ~
cd $HOME

# Go to specific directory
cd /var/log

# Go to previous directory
cd -

# Go up one level
cd ..

# Go up two levels
cd ../..

# Go to subdirectory
cd ./subdirectory
cd subdirectory  # Same as above

# Special directories
cd ~username     # Go to another user's home
cd /             # Go to root directory
```

**Pro Tips:**

```bash
# Create and change to directory in one command
mkdir -p ~/projects/newapp && cd $_

# Use CDPATH for quick navigation
export CDPATH=".:~:~/projects"
cd myproject  # Searches in CDPATH locations
```

### ls - List Directory Contents

```bash
# Basic listing
ls

# Long format (detailed)
ls -l
# drwxr-xr-x  5 user group 4096 Nov  7 10:30 folder
# -rw-r--r--  1 user group 1234 Nov  7 09:15 file.txt

# Show hidden files
ls -a
ls -la  # Long format + hidden files

# Human-readable sizes
ls -lh
# -rw-r--r--  1 user group 1.2K Nov  7 09:15 file.txt

# Sort by modification time
ls -lt

# Sort by size
ls -lS

# Reverse sort
ls -lr

# Recursive listing
ls -R

# List directories only
ls -d */

# Color output
ls --color=auto

# One file per line
ls -1
```

**Advanced ls Usage:**

```bash
# Show inode numbers
ls -i

# Show full timestamps
ls -l --time-style=full-iso

# List by extension
ls -X

# Tree-like format (requires tree command)
tree
tree -L 2  # Limit depth to 2 levels

# Combine multiple options
ls -lhaS  # Long, human-readable, all files, sorted by size
```

---

## File Operations

### Creating Files and Directories

```bash
# Create empty file
touch file.txt

# Create multiple files
touch file1.txt file2.txt file3.txt

# Create file with timestamp
touch file_$(date +%Y%m%d).txt

# Create directory
mkdir mydir

# Create nested directories
mkdir -p parent/child/grandchild

# Create multiple directories
mkdir dir1 dir2 dir3

# Create directory with permissions
mkdir -m 755 mydir
```

### Copying Files

```bash
# Copy file
cp source.txt destination.txt

# Copy to directory
cp file.txt /path/to/directory/

# Copy multiple files
cp file1.txt file2.txt /destination/

# Copy directory (recursive)
cp -r sourcedir/ destdir/

# Preserve attributes (permissions, timestamps)
cp -p file.txt copy.txt

# Interactive mode (ask before overwrite)
cp -i file.txt existing_file.txt

# Verbose mode
cp -v file.txt copy.txt

# Backup existing files
cp --backup=numbered file.txt existing.txt
```

### Moving and Renaming

```bash
# Rename file
mv oldname.txt newname.txt

# Move file to directory
mv file.txt /path/to/directory/

# Move multiple files
mv file1.txt file2.txt /destination/

# Move directory
mv olddir/ newdir/

# Interactive mode
mv -i source.txt destination.txt

# Force overwrite
mv -f source.txt destination.txt

# Update only (move if newer)
mv -u source.txt destination.txt
```

### Deleting Files

```bash
# Delete file
rm file.txt

# Delete multiple files
rm file1.txt file2.txt

# Delete directory (recursive)
rm -r directory/

# Force delete (no prompts)
rm -f file.txt

# Interactive mode (confirm each deletion)
rm -i file.txt

# Verbose mode
rm -v file.txt

# Delete empty directory
rmdir emptydir/

# Safe deletion (ask before deleting more than 3 files)
rm -I *.txt
```

**⚠️ Dangerous Commands - Use With Caution:**

```bash
# NEVER run these without being absolutely sure:
rm -rf /           # Deletes everything (requires --no-preserve-root)
rm -rf /*          # Deletes everything in root
rm -rf ~/*         # Deletes everything in home directory

# Safe practices:
# 1. Always use -i for interactive deletion
# 2. Test with ls first: ls -la path/to/delete
# 3. Use trash/move to temp instead of rm
```

### Viewing File Contents

```bash
# Display entire file
cat file.txt

# Display with line numbers
cat -n file.txt

# Display multiple files
cat file1.txt file2.txt

# Display first 10 lines
head file.txt

# Display first N lines
head -n 20 file.txt

# Display last 10 lines
tail file.txt

# Display last N lines
tail -n 20 file.txt

# Follow file in real-time (logs)
tail -f /var/log/syslog

# Display file page by page
less file.txt
more file.txt

# Search within less
# Press / then type search term
# Press n for next occurrence
# Press q to quit
```

### Searching for Files

```bash
# Find files by name
find . -name "*.txt"

# Find files case-insensitive
find . -iname "*.TXT"

# Find directories
find . -type d -name "mydir"

# Find files by size
find . -size +10M      # Larger than 10MB
find . -size -1k       # Smaller than 1KB

# Find files modified in last 7 days
find . -mtime -7

# Find and execute command
find . -name "*.log" -exec rm {} \;

# Find and delete
find . -name "*.tmp" -delete

# Locate (faster but requires updated database)
locate filename
sudo updatedb  # Update locate database
```

---

## File Permissions

### Understanding Permissions

```bash
# Permission format:
# -rwxrwxrwx
# ├┬──┬──┬──
# │└─ Owner permissions
# │  └─ Group permissions
# │    └─ Others permissions
# └─ File type: - (file), d (directory), l (link)

# Permission values:
# r (read)    = 4
# w (write)   = 2
# x (execute) = 1
# - (none)    = 0

# Common combinations:
# 755 = rwxr-xr-x (owner: all, group: rx, others: rx)
# 644 = rw-r--r-- (owner: rw, group: r, others: r)
# 700 = rwx------ (owner: all, group: none, others: none)
```

### chmod - Change Permissions

```bash
# Symbolic mode
chmod u+x script.sh          # Add execute for user
chmod g-w file.txt           # Remove write for group
chmod o+r file.txt           # Add read for others
chmod a+x script.sh          # Add execute for all

# Numeric mode
chmod 755 script.sh          # rwxr-xr-x
chmod 644 file.txt           # rw-r--r--
chmod 600 private.txt        # rw-------

# Recursive
chmod -R 755 directory/

# Set setuid/setgid
chmod u+s executable         # Set setuid
chmod g+s directory/         # Set setgid

# Sticky bit
chmod +t /tmp                # Set sticky bit
```

### chown - Change Ownership

```bash
# Change owner
sudo chown user file.txt

# Change owner and group
sudo chown user:group file.txt

# Recursive
sudo chown -R user:group directory/

# Change only group
sudo chown :group file.txt
# or
sudo chgrp group file.txt
```

### Viewing Permissions

```bash
# Detailed listing
ls -l file.txt
# -rw-r--r-- 1 user group 1234 Nov 7 10:30 file.txt

# Check if file is executable
test -x script.sh && echo "Executable" || echo "Not executable"

# Show numeric permissions
stat -c "%a %n" file.txt
# Output: 644 file.txt
```

---

## Process Management

### Viewing Processes

```bash
# Show current user processes
ps

# Show all processes
ps aux

# Show process tree
ps auxf
pstree

# Interactive process viewer
top

# Better alternative to top
htop  # Need to install: sudo apt install htop

# Show processes for specific user
ps -u username

# Search for process
ps aux | grep process_name
```

### Managing Processes

```bash
# Run process in background
command &

# Show background jobs
jobs

# Bring job to foreground
fg %1

# Send job to background
bg %1

# Suspend current process
Ctrl+Z

# Kill process by PID
kill 1234

# Force kill
kill -9 1234
kill -SIGKILL 1234

# Kill by name
killall process_name
pkill process_name

# Kill all user processes
killall -u username
```

### Process Priority

```bash
# Run with lower priority
nice -n 10 command

# Run with higher priority (requires sudo)
sudo nice -n -10 command

# Change priority of running process
renice -n 5 -p 1234

# Show process priority
ps -eo pid,nice,comm
```

---

## Input/Output Redirection

### Standard Streams

```
stdin  (0) - Standard Input
stdout (1) - Standard Output
stderr (2) - Standard Error
```

### Output Redirection

```bash
# Redirect stdout to file (overwrite)
command > output.txt

# Redirect stdout to file (append)
command >> output.txt

# Redirect stderr to file
command 2> errors.txt

# Redirect both stdout and stderr
command > output.txt 2>&1
command &> output.txt  # Bash shortcut

# Redirect stdout and stderr separately
command > output.txt 2> errors.txt

# Discard output
command > /dev/null

# Discard errors
command 2> /dev/null

# Discard everything
command &> /dev/null
```

### Input Redirection

```bash
# Read from file
command < input.txt

# Here document
cat << EOF > file.txt
Line 1
Line 2
Line 3
EOF

# Here string
grep "pattern" <<< "text to search"
```

### Pipes

```bash
# Send output of one command to another
ls -la | grep "txt"

# Chain multiple commands
ps aux | grep firefox | awk '{print $2}' | xargs kill

# Tee - write to file AND stdout
command | tee output.txt

# Append with tee
command | tee -a output.txt

# Write to multiple files
command | tee file1.txt file2.txt

# Example: save and display
ls -la | tee directory_listing.txt
```

### Practical Examples

```bash
# Count lines in file
wc -l < file.txt

# Sort and remove duplicates
sort file.txt | uniq > sorted.txt

# Find and count unique IPs in log
cat access.log | awk '{print $1}' | sort | uniq -c | sort -rn

# Monitor log in real-time and save
tail -f /var/log/syslog | tee system.log

# Pipeline with error handling
command 2>&1 | tee output.txt | grep ERROR
```

---

## Combining Commands

### Sequential Execution

```bash
# Run one after another (regardless of success)
command1 ; command2 ; command3

# Example
cd /tmp ; ls ; pwd
```

### Conditional Execution

```bash
# AND operator - run next only if previous succeeds
command1 && command2

# Example: compile and run
gcc program.c && ./a.out

# OR operator - run next only if previous fails
command1 || command2

# Example: create directory or exit
mkdir /tmp/test || exit 1

# Combined
command1 && command2 || command3
# If command1 succeeds, run command2
# If command1 fails, run command3
```

### Practical Combinations

```bash
# Update and install
sudo apt update && sudo apt upgrade -y

# Create directory and enter it
mkdir project && cd project

# Backup and modify
cp config.txt config.bak && vim config.txt

# Try multiple commands until one succeeds
command1 || command2 || command3 || echo "All failed"

# Deploy workflow
git pull && npm install && npm test && npm run build && pm2 restart app

# Cleanup on success
make && make test && make install && make clean
```

### Command Grouping

```bash
# Group commands in subshell (doesn't affect current shell)
(cd /tmp; ls; pwd)
pwd  # Still in original directory

# Group commands in current shell
{ cd /tmp; ls; pwd; }
pwd  # Now in /tmp

# Practical example: Transaction-like behavior
(
    cd /tmp || exit 1
    git clone repo || exit 1
    cd repo || exit 1
    make install || exit 1
) && echo "Success" || echo "Failed somewhere"
```

---

## Summary

### Key Commands Reference

| Category | Commands |
|----------|----------|
| **Navigation** | `pwd`, `cd`, `ls` |
| **Files** | `touch`, `cp`, `mv`, `rm`, `cat`, `head`, `tail`, `less` |
| **Search** | `find`, `locate`, `grep` |
| **Permissions** | `chmod`, `chown`, `chgrp` |
| **Processes** | `ps`, `top`, `kill`, `jobs`, `fg`, `bg` |
| **Redirection** | `>`, `>>`, `<`, `|`, `tee`, `2>`, `&>` |

### Redirection Quick Reference

```bash
>       # Redirect stdout (overwrite)
>>      # Redirect stdout (append)
2>      # Redirect stderr
&>      # Redirect both stdout and stderr
<       # Redirect stdin
|       # Pipe output to next command
tee     # Write to file and stdout
```

### Combining Commands

```bash
;       # Sequential execution
&&      # AND (run next if previous succeeds)
||      # OR (run next if previous fails)
|       # Pipe (send output to next command)
```

### Best Practices

1. ✅ Use `-i` flag for interactive deletion (`rm -i`)
2. ✅ Test with `ls` before `rm`
3. ✅ Use `&&` for dependent commands
4. ✅ Redirect errors when needed
5. ✅ Use absolute paths in scripts
6. ✅ Check command success with `$?`

### Common Patterns

```bash
# Create and enter directory
mkdir -p path/to/dir && cd $_

# Backup before modifying
cp file.txt{,.bak}  # Creates file.txt.bak

# Find and delete old files
find /tmp -type f -mtime +30 -delete

# Monitor log and save
tail -f /var/log/app.log | tee app_$(date +%Y%m%d).log

# Safe way to empty file
: > file.txt  # Better than cat /dev/null > file.txt
```

---

**Next Chapter:** [Working with Variables →](05-working-with-variables.md)

---

*Practice Exercise: Create a directory structure, copy some files, set permissions to 755, and practice piping commands like `ls -la | grep txt | wc -l`*
