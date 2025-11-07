# Chapter 16: Working with Arrays

## Introduction

Arrays are powerful data structures in Bash that allow you to store and manipulate collections of values. This chapter covers both indexed arrays (traditional numeric indices) and associative arrays (key-value pairs), providing the foundation for building complex scripts that handle multiple pieces of related data efficiently.

## Table of Contents
- [Indexed Arrays](#indexed-arrays)
- [Associative Arrays](#associative-arrays)
- [Array Operations](#array-operations)
- [Iterating Arrays](#iterating-arrays)
- [Multidimensional Arrays](#multidimensional-arrays)
- [Practical Use Cases](#practical-use-cases)

---

## Indexed Arrays

### Declaration and Initialization

```bash
# Method 1: Direct assignment
fruits[0]="apple"
fruits[1]="banana"
fruits[2]="cherry"

# Method 2: Declare and assign
declare -a fruits=("apple" "banana" "cherry")

# Method 3: Without declare
fruits=("apple" "banana" "cherry")

# Method 4: One at a time
fruits=()
fruits+=("apple")
fruits+=("banana")
fruits+=("cherry")

# Method 5: From command output
files=($(ls *.txt))
users=($(cut -d: -f1 /etc/passwd))

# Method 6: Sequence
numbers=({1..10})
letters=({a..z})
```

### Accessing Array Elements

```bash
fruits=("apple" "banana" "cherry" "date")

# Access single element
echo "${fruits[0]}"          # apple
echo "${fruits[2]}"          # cherry
echo "${fruits[-1]}"         # date (last element)

# Access all elements
echo "${fruits[@]}"          # apple banana cherry date
echo "${fruits[*]}"          # apple banana cherry date

# Number of elements
echo "${#fruits[@]}"         # 4

# Indices
echo "${!fruits[@]}"         # 0 1 2 3

# Length of specific element
echo "${#fruits[0]}"         # 5 (length of "apple")
```

### Array Slicing

```bash
fruits=("apple" "banana" "cherry" "date" "elderberry")

# Slice: ${array[@]:start:length}
echo "${fruits[@]:1:3}"      # banana cherry date
echo "${fruits[@]:2}"        # cherry date elderberry

# Copy array
backup=("${fruits[@]}")

# Subset of array
subset=("${fruits[@]:1:2}")  # banana cherry
```

### Modifying Arrays

```bash
fruits=("apple" "banana" "cherry")

# Append
fruits+=("date")
fruits[${#fruits[@]}]="elderberry"

# Insert at beginning
fruits=("mango" "${fruits[@]}")

# Replace element
fruits[1]="blueberry"

# Delete element
unset fruits[1]
# Note: This creates a gap in indices

# Delete entire array
unset fruits

# Reassign
fruits=("${fruits[@]}" "grape")  # Add to end
```

### Array Patterns

```bash
# Remove pattern from all elements
paths=("/home/user/file1.txt" "/home/user/file2.txt")
basenames=("${paths[@]##*/}")     # file1.txt file2.txt

# Replace pattern in all elements
files=("test.old" "data.old" "config.old")
new_files=("${files[@]/.old/.new}")

# Uppercase all elements
names=("alice" "bob" "charlie")
upper_names=("${names[@]^^}")
```

---

## Associative Arrays

### Declaration and Usage

```bash
# Must declare as associative
declare -A user_info

# Assignment
user_info[name]="Alice"
user_info[age]=25
user_info[city]="New York"

# Or initialize at declaration
declare -A config=(
    [host]="localhost"
    [port]=8080
    [debug]=true
)

# Access
echo "${user_info[name]}"        # Alice
echo "${config[port]}"           # 8080

# All keys
echo "${!user_info[@]}"          # name age city

# All values
echo "${user_info[@]}"           # Alice 25 New York

# Check if key exists
[[ -v user_info[email] ]] && echo "Email exists" || echo "No email"
```

### Working with Associative Arrays

```bash
declare -A database

# Add entries
database[db_host]="localhost"
database[db_port]=3306
database[db_name]="myapp"
database[db_user]="admin"

# Update
database[db_port]=3307

# Delete key
unset database[db_user]

# Iterate keys
for key in "${!database[@]}"; do
    echo "$key = ${database[$key]}"
done

# Count entries
echo "Total: ${#database[@]}"

# Clear all
unset database
declare -A database
```

---

## Array Operations

### Sorting Arrays

```bash
# Sort indexed array
fruits=("cherry" "apple" "banana" "date")

# Method 1: Using sort command
IFS=$'\n' sorted=($(sort <<<"${fruits[*]}"))
unset IFS

# Method 2: Bubble sort function
bubble_sort() {
    local -n arr=$1
    local n=${#arr[@]}
    
    for ((i=0; i<n-1; i++)); do
        for ((j=0; j<n-i-1; j++)); do
            if [[ "${arr[j]}" > "${arr[j+1]}" ]]; then
                # Swap
                local temp="${arr[j]}"
                arr[j]="${arr[j+1]}"
                arr[j+1]="$temp"
            fi
        done
    done
}

fruits=("cherry" "apple" "banana")
bubble_sort fruits
echo "${fruits[@]}"  # apple banana cherry

# Numeric sort
numbers=(5 2 8 1 9 3)
IFS=$'\n' sorted=($(sort -n <<<"${numbers[*]}"))
unset IFS
```

### Searching Arrays

```bash
# Check if value exists
contains() {
    local value="$1"
    shift
    local arr=("$@")
    
    for item in "${arr[@]}"; do
        [[ "$item" == "$value" ]] && return 0
    done
    return 1
}

fruits=("apple" "banana" "cherry")
if contains "banana" "${fruits[@]}"; then
    echo "Found banana"
fi

# Find index of value
find_index() {
    local value="$1"
    shift
    local arr=("$@")
    
    for i in "${!arr[@]}"; do
        if [[ "${arr[$i]}" == "$value" ]]; then
            echo "$i"
            return 0
        fi
    done
    return 1
}

index=$(find_index "banana" "${fruits[@]}")
echo "Index: $index"
```

### Array Comparison

```bash
# Compare two arrays
arrays_equal() {
    local -n arr1=$1
    local -n arr2=$2
    
    # Different lengths
    [[ ${#arr1[@]} -ne ${#arr2[@]} ]] && return 1
    
    # Compare elements
    for i in "${!arr1[@]}"; do
        [[ "${arr1[$i]}" != "${arr2[$i]}" ]] && return 1
    done
    
    return 0
}

array1=(1 2 3)
array2=(1 2 3)
array3=(1 2 4)

arrays_equal array1 array2 && echo "Equal" || echo "Different"
arrays_equal array1 array3 && echo "Equal" || echo "Different"
```

### Merging Arrays

```bash
# Concatenate arrays
arr1=(1 2 3)
arr2=(4 5 6)
merged=("${arr1[@]}" "${arr2[@]}")
echo "${merged[@]}"  # 1 2 3 4 5 6

# Unique merge
unique_merge() {
    local -n result=$1
    shift
    declare -A seen
    
    for arr in "$@"; do
        eval "local temp=(\"\${${arr}[@]}\")"
        for item in "${temp[@]}"; do
            if [[ -z "${seen[$item]}" ]]; then
                result+=("$item")
                seen[$item]=1
            fi
        done
    done
}

# Remove duplicates from array
remove_duplicates() {
    local -n arr=$1
    local unique=()
    declare -A seen
    
    for item in "${arr[@]}"; do
        if [[ -z "${seen[$item]}" ]]; then
            unique+=("$item")
            seen[$item]=1
        fi
    done
    
    arr=("${unique[@]}")
}

numbers=(1 2 3 2 4 3 5)
remove_duplicates numbers
echo "${numbers[@]}"  # 1 2 3 4 5
```

---

## Iterating Arrays

### For Loops

```bash
fruits=("apple" "banana" "cherry")

# Iterate values
for fruit in "${fruits[@]}"; do
    echo "Fruit: $fruit"
done

# Iterate indices
for i in "${!fruits[@]}"; do
    echo "Index $i: ${fruits[$i]}"
done

# C-style loop
for ((i=0; i<${#fruits[@]}; i++)); do
    echo "Element $i: ${fruits[$i]}"
done
```

### While Loops

```bash
fruits=("apple" "banana" "cherry")
i=0

while [ $i -lt ${#fruits[@]} ]; do
    echo "${fruits[$i]}"
    ((i++))
done

# With read (for associative arrays)
declare -A config=(
    [host]="localhost"
    [port]=8080
)

while IFS= read -r key; do
    echo "$key: ${config[$key]}"
done < <(printf '%s\n' "${!config[@]}")
```

### Parallel Iteration

```bash
# Iterate two arrays together
names=("Alice" "Bob" "Charlie")
ages=(25 30 35)

for i in "${!names[@]}"; do
    echo "${names[$i]} is ${ages[$i]} years old"
done
```

---

## Multidimensional Arrays

### Simulating 2D Arrays

```bash
# Method 1: Using naming convention
matrix_0_0=1
matrix_0_1=2
matrix_1_0=3
matrix_1_1=4

# Method 2: Using associative array
declare -A matrix
matrix[0,0]=1
matrix[0,1]=2
matrix[1,0]=3
matrix[1,1]=4

# Access
echo "${matrix[0,0]}"  # 1
echo "${matrix[1,1]}"  # 4

# Iterate 2D array
for i in {0..1}; do
    for j in {0..1}; do
        echo "matrix[$i,$j] = ${matrix[$i,$j]}"
    done
done

# Function to print matrix
print_matrix() {
    local -n mat=$1
    local rows=$2
    local cols=$3
    
    for ((i=0; i<rows; i++)); do
        for ((j=0; j<cols; j++)); do
            printf "%3s " "${mat[$i,$j]}"
        done
        echo
    done
}

declare -A grid
grid[0,0]=1; grid[0,1]=2; grid[0,2]=3
grid[1,0]=4; grid[1,1]=5; grid[1,2]=6
grid[2,0]=7; grid[2,1]=8; grid[2,2]=9

print_matrix grid 3 3
```

---

## Practical Use Cases

### Menu System

```bash
#!/bin/bash

# Menu with array
declare -a menu_options=(
    "Deploy Application"
    "Check Status"
    "View Logs"
    "Restart Service"
    "Quit"
)

display_menu() {
    echo "=== Main Menu ==="
    for i in "${!menu_options[@]}"; do
        echo "$((i+1)). ${menu_options[$i]}"
    done
}

while true; do
    display_menu
    read -p "Select option: " choice
    
    # Validate input
    if [[ ! "$choice" =~ ^[0-9]+$ ]] || [ "$choice" -lt 1 ] || [ "$choice" -gt ${#menu_options[@]} ]; then
        echo "Invalid option"
        continue
    fi
    
    # Execute based on choice
    case $choice in
        1) echo "Deploying...";;
        2) echo "Checking status...";;
        3) echo "Viewing logs...";;
        4) echo "Restarting...";;
        5) echo "Goodbye!"; break;;
    esac
    
    echo
done
```

### Configuration Management

```bash
#!/bin/bash

# Parse INI file into associative array
declare -A config

parse_ini() {
    local file="$1"
    local section=""
    
    while IFS='=' read -r key value; do
        # Skip comments and empty lines
        [[ "$key" =~ ^[[:space:]]*# ]] && continue
        [[ -z "$key" ]] && continue
        
        # Section headers
        if [[ "$key" =~ ^\[(.*)\] ]]; then
            section="${BASH_REMATCH[1]}"
            continue
        fi
        
        # Remove whitespace
        key=$(echo "$key" | xargs)
        value=$(echo "$value" | xargs)
        
        # Store with section prefix
        if [[ -n "$section" ]]; then
            config["${section}.${key}"]="$value"
        else
            config["$key"]="$value"
        fi
    done < "$file"
}

# Usage
cat > config.ini << 'EOF'
[database]
host=localhost
port=3306
name=myapp

[cache]
enabled=true
ttl=3600
EOF

parse_ini config.ini

# Access configuration
echo "DB Host: ${config[database.host]}"
echo "DB Port: ${config[database.port]}"
echo "Cache TTL: ${config[cache.ttl]}"
```

### Inventory System

```bash
#!/bin/bash

# Product inventory with associative arrays
declare -A inventory

# Add products
add_product() {
    local id="$1"
    local name="$2"
    local quantity="$3"
    local price="$4"
    
    inventory["${id}.name"]="$name"
    inventory["${id}.quantity"]="$quantity"
    inventory["${id}.price"]="$price"
}

# Update quantity
update_quantity() {
    local id="$1"
    local change="$2"
    
    local current="${inventory[${id}.quantity]}"
    inventory["${id}.quantity"]=$((current + change))
}

# Display inventory
display_inventory() {
    echo "=== Inventory ==="
    printf "%-10s %-20s %-10s %-10s\n" "ID" "Name" "Quantity" "Price"
    echo "-----------------------------------------------------------"
    
    # Get unique product IDs
    declare -A products
    for key in "${!inventory[@]}"; do
        local id="${key%%.*}"
        products[$id]=1
    done
    
    # Display each product
    for id in "${!products[@]}"; do
        printf "%-10s %-20s %-10s $%-9.2f\n" \
            "$id" \
            "${inventory[${id}.name]}" \
            "${inventory[${id}.quantity]}" \
            "${inventory[${id}.price]}"
    done
}

# Example usage
add_product "P001" "Laptop" 10 999.99
add_product "P002" "Mouse" 50 29.99
add_product "P003" "Keyboard" 30 79.99

display_inventory

update_quantity "P001" -2
update_quantity "P002" 10

echo
display_inventory
```

### Word Frequency Counter

```bash
#!/bin/bash

# Count word frequency using associative array
count_words() {
    local file="$1"
    declare -A word_count
    
    # Read and process file
    while read -r line; do
        # Convert to lowercase and split into words
        for word in $(echo "$line" | tr '[:upper:]' '[:lower:]' | tr -s '[:space:]' '\n' | grep -v '^$'); do
            # Remove punctuation
            word="${word//[^a-z0-9]/}"
            [[ -z "$word" ]] && continue
            
            # Count
            ((word_count[$word]++))
        done
    done < "$file"
    
    # Display results sorted by frequency
    for word in "${!word_count[@]}"; do
        echo "${word_count[$word]} $word"
    done | sort -rn | head -20
}

# Usage
cat > sample.txt << 'EOF'
The quick brown fox jumps over the lazy dog.
The dog was really lazy.
EOF

echo "Top 20 words:"
count_words sample.txt
```

### Task Queue

```bash
#!/bin/bash

# Simple task queue using arrays
declare -a task_queue
declare -A task_status

# Add task
add_task() {
    local task="$1"
    task_queue+=("$task")
    task_status["$task"]="pending"
    echo "Added task: $task"
}

# Process next task
process_next_task() {
    if [ ${#task_queue[@]} -eq 0 ]; then
        echo "No tasks in queue"
        return 1
    fi
    
    local task="${task_queue[0]}"
    task_queue=("${task_queue[@]:1}")  # Remove first element
    
    echo "Processing: $task"
    task_status["$task"]="processing"
    
    # Simulate work
    sleep 1
    
    task_status["$task"]="completed"
    echo "Completed: $task"
}

# Show queue status
show_queue() {
    echo "=== Task Queue ==="
    echo "Pending tasks: ${#task_queue[@]}"
    
    for task in "${task_queue[@]}"; do
        echo "  - $task [${task_status[$task]}]"
    done
    
    echo
    echo "=== All Tasks ==="
    for task in "${!task_status[@]}"; do
        echo "  $task: ${task_status[$task]}"
    done
}

# Example usage
add_task "Backup database"
add_task "Deploy application"
add_task "Send notification"

show_queue
echo

process_next_task
process_next_task

echo
show_queue
```

### CSV to Array Converter

```bash
#!/bin/bash

# Read CSV into array of associative arrays
declare -a records

read_csv() {
    local file="$1"
    local -a headers
    local line_num=0
    
    while IFS=, read -r -a fields; do
        if [ $line_num -eq 0 ]; then
            # First line is headers
            headers=("${fields[@]}")
        else
            # Create associative array for this record
            declare -A record
            for i in "${!headers[@]}"; do
                local key="${headers[$i]}"
                local value="${fields[$i]}"
                record["$key"]="$value"
            done
            
            # Store record (convert to string representation)
            local record_str=""
            for key in "${!record[@]}"; do
                record_str+="$key=${record[$key]};"
            done
            records+=("$record_str")
        fi
        ((line_num++))
    done < "$file"
}

# Example CSV
cat > data.csv << 'EOF'
name,age,city
Alice,25,New York
Bob,30,London
Charlie,35,Tokyo
EOF

read_csv data.csv

echo "Loaded ${#records[@]} records"
for record in "${records[@]}"; do
    echo "$record"
done
```

---

## Summary

In this chapter, you learned:

- ✅ **Indexed Arrays**: Traditional arrays with numeric indices
- ✅ **Associative Arrays**: Key-value pairs (hash maps)
- ✅ **Operations**: Sorting, searching, merging, filtering
- ✅ **Iteration**: Multiple ways to loop through arrays
- ✅ **Advanced**: Multidimensional array simulation
- ✅ **Practical**: Real-world applications and patterns

### Key Takeaways

| Array Type | Use Case | Syntax |
|------------|----------|--------|
| Indexed | Lists, sequences | `arr=(a b c)` |
| Associative | Key-value data | `declare -A arr` |
| Access | Get element | `${arr[index]}` |
| Length | Count elements | `${#arr[@]}` |
| Iterate | Loop values | `for x in "${arr[@]}"` |
| Keys | Get indices/keys | `${!arr[@]}` |

### Next Steps

In [Chapter 17: String Manipulation](./17-string-manipulation.md), you'll master advanced string operations and parameter expansion techniques.

---

## Practice Exercises

1. Create a phonebook system using associative arrays
2. Implement a simple database with add, update, delete, and search functions
3. Build a task manager with priority queue functionality
4. Write a script to analyze and report on CSV data
5. Create a caching system using arrays to store frequently accessed data

---

**Next Chapter:** [17 - String Manipulation](17-string-manipulation.md)
