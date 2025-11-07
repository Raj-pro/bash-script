# Chapter 34: Optimizing Bash Performance

## Introduction

Performance optimization ensures scripts run efficiently. This chapter covers profiling, optimization techniques, and performance best practices.

---

## Profiling Scripts

### Time Measurement

```bash
#!/bin/bash

# Time entire script
time ./script.sh

# Time specific section
start_time=$(date +%s)
# Your code here
end_time=$(date +%s)
duration=$((end_time - start_time))
echo "Duration: ${duration}s"

# Using SECONDS variable
SECONDS=0
# Your code here
echo "Duration: ${SECONDS}s"
```

### Detailed Profiling

```bash
#!/bin/bash
# Enable profiling
PS4='+ $(date +%s.%N)\t'
set -x

# Your code here

set +x
```

---

## Avoiding Subshells

### Inefficient (Creates Subshells)

```bash
# BAD: Subshell for each file
for file in *.txt; do
    count=$(wc -l < "$file")
    echo "$file: $count"
done

# BAD: Unnecessary cat
cat file.txt | grep "pattern"

# BAD: Command substitution in loop
for i in {1..1000}; do
    result=$(some_command)
done
```

### Optimized (Avoids Subshells)

```bash
# GOOD: Use built-in functionality
wc -l *.txt

# GOOD: Direct redirection
grep "pattern" < file.txt

# GOOD: Process once
result=$(some_command)
for i in {1..1000}; do
    use "$result"
done
```

---

## Efficient Loops

### Loop Optimization

```bash
# SLOW: External command per iteration
for file in *.txt; do
    if grep -q "pattern" "$file"; then
        echo "$file"
    fi
done

# FAST: Single grep command
grep -l "pattern" *.txt

# SLOW: Multiple process spawns
for i in {1..1000}; do
    echo "$i" | awk '{print $1 * 2}'
done

# FAST: Let awk handle the loop
awk 'BEGIN {for(i=1; i<=1000; i++) print i*2}'
```

---

## String Operations

### Efficient String Manipulation

```bash
# SLOW: Using sed for simple replacement
result=$(echo "$string" | sed 's/old/new/')

# FAST: Use parameter expansion
result="${string/old/new}"

# SLOW: Multiple sed calls
echo "$string" | sed 's/a/b/' | sed 's/c/d/'

# FAST: Single sed with multiple expressions
echo "$string" | sed 's/a/b/; s/c/d/'

# FAST: Parameter expansion for substrings
text="Hello World"
${text:0:5}     # "Hello"
${text:6}       # "World"
${text#Hello }  # "World"
${text%World}   # "Hello "
```

---

## File Processing

### Read Files Efficiently

```bash
# SLOW: Read line by line with subshells
cat file.txt | while read line; do
    process "$line"
done

# FAST: Direct input redirection
while IFS= read -r line; do
    process "$line"
done < file.txt

# FAST: Process large files in chunks
xargs -n 100 -P 4 process < file.txt
```

### Parallel Processing

```bash
#!/bin/bash

# Process files in parallel
MAX_JOBS=4

process_file() {
    local file=$1
    # Processing logic
    gzip "$file"
}

# Export function for parallel execution
export -f process_file

# Find and process in parallel
find . -name "*.txt" | xargs -n 1 -P $MAX_JOBS -I {} bash -c 'process_file "$@"' _ {}
```

---

## Array Operations

### Efficient Array Usage

```bash
# Building arrays efficiently
# SLOW: Append in loop
for i in {1..1000}; do
    array+=("$i")
done

# FAST: Array initialization
array=({1..1000})

# Iterating arrays
# SLOW: Index-based
for i in ${!array[@]}; do
    echo "${array[$i]}"
done

# FAST: Direct iteration
for item in "${array[@]}"; do
    echo "$item"
done
```

---

## Command Substitution

### Optimization Techniques

```bash
# SLOW: Nested command substitution
result=$(echo $(cat file.txt))

# FAST: Direct read
result=$(< file.txt)

# SLOW: Multiple command substitutions
var1=$(command1)
var2=$(command2)
var3=$(command3)

# FAST: Single call with multiple outputs
IFS=' ' read -r var1 var2 var3 < <(paste <(command1) <(command2) <(command3))
```

---

## Disk I/O Optimization

### Minimize Disk Access

```bash
# SLOW: Multiple writes
for i in {1..1000}; do
    echo "$i" >> file.txt
done

# FAST: Single write
{
    for i in {1..1000}; do
        echo "$i"
    done
} > file.txt

# FAST: Use tmpfs for temporary files
temp_file=/dev/shm/temp.$$
trap "rm -f '$temp_file'" EXIT
```

---

## Network Operations

### Efficient Network Calls

```bash
# SLOW: Multiple API calls
for id in {1..100}; do
    curl "https://api.example.com/user/$id"
done

# FAST: Batch API call if supported
curl "https://api.example.com/users?ids=1-100"

# FAST: Parallel requests with limits
parallel_curl() {
    local url=$1
    curl -s "$url"
}

export -f parallel_curl
seq 1 100 | xargs -n 1 -P 10 -I {} bash -c 'parallel_curl "https://api.example.com/user/{}"'
```

---

## Memory Management

### Reduce Memory Usage

```bash
# SLOW: Load entire file into memory
content=$(cat large_file.txt)
process "$content"

# FAST: Stream processing
while IFS= read -r line; do
    process "$line"
done < large_file.txt

# SLOW: Large array in memory
mapfile -t lines < large_file.txt

# FAST: Process on-demand
process_line() {
    # Process one line at a time
    echo "$1"
}

while IFS= read -r line; do
    process_line "$line"
done < large_file.txt
```

---

## Built-in vs External Commands

### Use Built-ins When Possible

```bash
# SLOW: External commands
count=$(echo "$var" | wc -w)
substring=$(echo "$var" | cut -c 1-5)

# FAST: Bash built-ins
count=($var)
count=${#count[@]}
substring=${var:0:5}

# Built-in comparisons
# SLOW
if [ $(echo "$a > $b" | bc) -eq 1 ]; then

# FAST
if (( a > b )); then
```

---

## Caching Results

### Cache Expensive Operations

```bash
#!/bin/bash

declare -A cache

expensive_operation() {
    local key=$1
    
    # Check cache
    if [ -n "${cache[$key]}" ]; then
        echo "${cache[$key]}"
        return 0
    fi
    
    # Perform expensive operation
    local result=$(heavy_computation "$key")
    
    # Store in cache
    cache[$key]=$result
    
    echo "$result"
}

# Usage
for i in {1..100}; do
    expensive_operation "key_$((i % 10))"  # Only 10 actual computations
done
```

---

## Benchmarking Script

### Performance Comparison Tool

```bash
#!/bin/bash

benchmark() {
    local name=$1
    local iterations=${2:-100}
    shift 2
    local command="$@"
    
    echo "Benchmarking: $name"
    
    SECONDS=0
    for ((i=0; i<iterations; i++)); do
        eval "$command" > /dev/null 2>&1
    done
    
    echo "  Time: ${SECONDS}s for $iterations iterations"
    echo "  Average: $(bc <<< "scale=4; $SECONDS / $iterations")s per iteration"
    echo
}

# Example usage
benchmark "External wc" 1000 'wc -l < /etc/passwd'
benchmark "Parameter expansion" 1000 'var="/etc/passwd"; ${var//[^:]}'
```

---

## Performance Checklist

- [ ] Avoid unnecessary subshells
- [ ] Use built-in commands over external
- [ ] Minimize disk I/O
- [ ] Process files in parallel when possible
- [ ] Use parameter expansion for strings
- [ ] Cache expensive computations
- [ ] Stream large files instead of loading into memory
- [ ] Batch network requests
- [ ] Profile before optimizing
- [ ] Test performance improvements

---

## Summary

- Profile scripts to identify bottlenecks
- Avoid subshells and external commands
- Use built-in Bash features
- Implement parallel processing
- Cache expensive operations
- Stream large files
- Minimize disk and network I/O

---

**Next Chapter:** [35 - Testing Bash Scripts](35-testing-bash-scripts.md)
