# Chapter 20: Working with Processes

## Introduction

Process management is essential for system automation and scripting. This chapter covers how to manage, monitor, and control processes from Bash scripts.

---

## Understanding Processes

### What is a Process?

A process is a running instance of a program with:
- **PID** (Process ID): Unique identifier
- **PPID** (Parent Process ID): ID of parent process
- **State**: Running, sleeping, stopped, zombie
- **Resources**: Memory, CPU, file descriptors

### Viewing Processes

```bash
# List all processes
ps aux

# Process tree
pstree

# Real-time process viewer
top
htop  # More user-friendly (may need installation)

# Specific process
ps -p PID

# Processes by user
ps -u username
```

---

## Process States

```bash
# Common process states (ps output)
R  # Running or runnable
S  # Sleeping (waiting for event)
D  # Uninterruptible sleep (usually I/O)
T  # Stopped (job control)
Z  # Zombie (terminated but not reaped)

# Check process state
ps -o stat,pid,cmd -p PID
```

---

## Background and Foreground Jobs

### Running Jobs in Background

```bash
# Run command in background with &
sleep 100 &

# Check background jobs
jobs

# Output:
# [1]+ Running   sleep 100 &
```

### Job Control

```bash
# Run in foreground
sleep 100

# Suspend with Ctrl+Z
# [1]+ Stopped    sleep 100

# Resume in background
bg %1

# Resume in foreground
fg %1

# List jobs
jobs -l  # Include PID

# Kill specific job
kill %1
```

### Practical Background Job Example

```bash
#!/bin/bash

# Start long-running process
./long_process.sh &
PROCESS_PID=$!

echo "Started process with PID: $PROCESS_PID"

# Do other work
echo "Doing other tasks..."

# Wait for background process
wait $PROCESS_PID
echo "Background process completed"
```

---

## Process Control with Signals

### Common Signals

```bash
SIGTERM (15)  # Terminate gracefully (default kill signal)
SIGKILL (9)   # Force kill (cannot be caught)
SIGINT (2)    # Interrupt (Ctrl+C)
SIGHUP (1)    # Hangup
SIGSTOP (19)  # Stop process (cannot be caught)
SIGCONT (18)  # Continue stopped process
SIGUSR1 (10)  # User-defined signal 1
SIGUSR2 (12)  # User-defined signal 2

# List all signals
kill -l
```

### Sending Signals

```bash
# Terminate process (graceful)
kill PID
kill -15 PID
kill -TERM PID

# Force kill
kill -9 PID
kill -KILL PID

# Send signal to process group
kill -TERM -PGID

# Kill process by name
pkill process_name
killall process_name
```

### Signal Handling in Scripts

```bash
#!/bin/bash

# Trap signals
cleanup() {
    echo "Cleaning up..."
    # Cleanup code
    exit 0
}

trap cleanup SIGTERM SIGINT

echo "Running... (Ctrl+C to stop)"
while true; do
    sleep 1
    echo "Working..."
done
```

---

## Managing Process Priority

### nice and renice

```bash
# nice values: -20 (highest) to 19 (lowest)
# Default is 0

# Start with lower priority
nice -n 10 ./cpu_intensive.sh

# Start with higher priority (requires sudo)
sudo nice -n -10 ./important.sh

# Change priority of running process
renice -n 5 -p PID

# By user
sudo renice -n 10 -u username
```

### ionice (I/O Priority)

```bash
# I/O scheduling classes:
# 1: Real-time (highest)
# 2: Best-effort (default)
# 3: Idle (lowest)

# Set I/O priority
ionice -c 3 ./backup.sh

# Set class and priority level (0-7)
ionice -c 2 -n 7 ./large_copy.sh
```

---

## Process Monitoring in Scripts

### Checking if Process is Running

```bash
#!/bin/bash

is_running() {
    local pid=$1
    kill -0 "$pid" 2>/dev/null
    return $?
}

# Usage
if is_running 1234; then
    echo "Process 1234 is running"
else
    echo "Process 1234 is not running"
fi
```

### Finding Process by Name

```bash
#!/bin/bash

get_pid() {
    local process_name=$1
    pgrep -x "$process_name"
}

# Usage
PID=$(get_pid "nginx")
if [ -n "$PID" ]; then
    echo "nginx is running with PID: $PID"
else
    echo "nginx is not running"
fi
```

### Monitoring Process Resources

```bash
#!/bin/bash

monitor_process() {
    local pid=$1
    local interval=5
    
    while kill -0 "$pid" 2>/dev/null; do
        # Get CPU and memory usage
        ps -p "$pid" -o %cpu,%mem,vsz,rss,etime
        sleep $interval
    done
}

# Usage
monitor_process 1234
```

---

## nohup and disown

### nohup (No Hangup)

Continues running after logout:

```bash
# Run command that survives logout
nohup ./long_script.sh &

# Output goes to nohup.out by default
# Redirect output
nohup ./script.sh > output.log 2>&1 &
```

### disown

Removes job from shell's job table:

```bash
# Start process
./long_script.sh &

# Disown it (survives shell exit)
disown %1

# Disown all jobs
disown -a

# List jobs before disown
jobs
disown %1
jobs  # Job no longer listed
```

### Comparison

```bash
# nohup: Start detached from terminal
nohup command &

# disown: Detach already running job
command &
disown

# Both: Maximum protection
nohup command > /dev/null 2>&1 &
disown
```

---

## Process Synchronization

### wait Command

```bash
#!/bin/bash

# Wait for specific process
sleep 10 &
PID=$!
wait $PID
echo "Process $PID completed"

# Wait for all background jobs
sleep 5 &
sleep 10 &
sleep 15 &
wait
echo "All processes completed"
```

### Parallel Processing

```bash
#!/bin/bash

# Run multiple tasks in parallel
process_file() {
    local file=$1
    echo "Processing $file..."
    sleep 2
    echo "$file done"
}

# Process files in parallel
for file in file1.txt file2.txt file3.txt; do
    process_file "$file" &
done

# Wait for all to complete
wait
echo "All files processed"
```

### Limited Parallelism

```bash
#!/bin/bash

MAX_JOBS=3

# Function to limit concurrent jobs
wait_for_jobs() {
    while [ $(jobs -r | wc -l) -ge $MAX_JOBS ]; do
        sleep 1
    done
}

# Process with limit
for i in {1..10}; do
    wait_for_jobs
    echo "Starting job $i"
    sleep 5 &
done

wait
echo "All jobs completed"
```

---

## Process Substitution

### Basic Process Substitution

```bash
# <(command) - Treat command output as file
diff <(ls dir1) <(ls dir2)

# Compare sorted lists
diff <(sort file1.txt) <(sort file2.txt)

# Multiple process substitutions
paste <(cut -d: -f1 /etc/passwd) <(cut -d: -f3 /etc/passwd)
```

### Practical Examples

```bash
# Compare running processes on two systems
diff <(ssh server1 'ps aux | sort') <(ssh server2 'ps aux | sort')

# Merge multiple log files by timestamp
sort -m <(sort log1.txt) <(sort log2.txt) <(sort log3.txt)

# Process data pipeline
while read line; do
    echo "Processing: $line"
done < <(find /data -name "*.txt" -mtime -1)
```

---

## Monitoring System Processes

### CPU Usage

```bash
#!/bin/bash

# Top CPU consuming processes
ps aux --sort=-%cpu | head -n 10

# Specific command CPU usage
ps aux | grep nginx | awk '{sum+=$3} END {print "Total CPU:", sum"%"}'
```

### Memory Usage

```bash
#!/bin/bash

# Top memory consuming processes
ps aux --sort=-%mem | head -n 10

# Memory by command
ps -eo comm,rss | awk '{mem[$1]+=$2} END {for (cmd in mem) print cmd, mem[cmd]/1024"MB"}' | sort -k2 -rn
```

### Process Count Monitoring

```bash
#!/bin/bash

# Monitor process count
watch_process_count() {
    local max_count=$1
    local process_name=$2
    
    while true; do
        count=$(pgrep -c "$process_name")
        echo "$(date): $process_name count: $count"
        
        if [ $count -gt $max_count ]; then
            echo "WARNING: Too many $process_name processes!" >&2
            # Send alert
        fi
        
        sleep 60
    done
}

# Usage
watch_process_count 10 "httpd"
```

---

## Practical Examples

### Process Manager Script

```bash
#!/bin/bash

PROCESS_NAME="myapp"
PIDFILE="/var/run/${PROCESS_NAME}.pid"

start() {
    if [ -f "$PIDFILE" ]; then
        PID=$(cat "$PIDFILE")
        if kill -0 "$PID" 2>/dev/null; then
            echo "$PROCESS_NAME is already running (PID: $PID)"
            return 1
        fi
    fi
    
    echo "Starting $PROCESS_NAME..."
    /usr/bin/$PROCESS_NAME &
    echo $! > "$PIDFILE"
    echo "$PROCESS_NAME started (PID: $(cat $PIDFILE))"
}

stop() {
    if [ ! -f "$PIDFILE" ]; then
        echo "$PROCESS_NAME is not running"
        return 1
    fi
    
    PID=$(cat "$PIDFILE")
    echo "Stopping $PROCESS_NAME (PID: $PID)..."
    
    kill "$PID"
    
    # Wait for graceful shutdown
    for i in {1..10}; do
        if ! kill -0 "$PID" 2>/dev/null; then
            rm -f "$PIDFILE"
            echo "$PROCESS_NAME stopped"
            return 0
        fi
        sleep 1
    done
    
    # Force kill if still running
    echo "Force killing $PROCESS_NAME..."
    kill -9 "$PID"
    rm -f "$PIDFILE"
}

status() {
    if [ ! -f "$PIDFILE" ]; then
        echo "$PROCESS_NAME is not running"
        return 1
    fi
    
    PID=$(cat "$PIDFILE")
    if kill -0 "$PID" 2>/dev/null; then
        echo "$PROCESS_NAME is running (PID: $PID)"
        ps -p "$PID" -o %cpu,%mem,etime
        return 0
    else
        echo "$PROCESS_NAME is not running (stale PID file)"
        rm -f "$PIDFILE"
        return 1
    fi
}

case "$1" in
    start)
        start
        ;;
    stop)
        stop
        ;;
    restart)
        stop
        sleep 2
        start
        ;;
    status)
        status
        ;;
    *)
        echo "Usage: $0 {start|stop|restart|status}"
        exit 1
        ;;
esac
```

### Resource Monitor

```bash
#!/bin/bash

THRESHOLD_CPU=80
THRESHOLD_MEM=90
LOG_FILE="/var/log/resource_monitor.log"

log_alert() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

check_resources() {
    # Check CPU
    while IFS= read -r line; do
        pid=$(echo "$line" | awk '{print $2}')
        cpu=$(echo "$line" | awk '{print $3}')
        cmd=$(echo "$line" | awk '{print $11}')
        
        if (( $(echo "$cpu > $THRESHOLD_CPU" | bc -l) )); then
            log_alert "HIGH CPU: $cmd (PID: $pid) using ${cpu}%"
        fi
    done < <(ps aux --sort=-%cpu | head -n 20)
    
    # Check Memory
    while IFS= read -r line; do
        pid=$(echo "$line" | awk '{print $2}')
        mem=$(echo "$line" | awk '{print $4}')
        cmd=$(echo "$line" | awk '{print $11}')
        
        if (( $(echo "$mem > $THRESHOLD_MEM" | bc -l) )); then
            log_alert "HIGH MEM: $cmd (PID: $pid) using ${mem}%"
        fi
    done < <(ps aux --sort=-%mem | head -n 20)
}

# Run continuously
while true; do
    check_resources
    sleep 300  # Check every 5 minutes
done
```

### Parallel Task Processor

```bash
#!/bin/bash

MAX_PARALLEL=4
TASK_LIST="tasks.txt"

process_task() {
    local task=$1
    echo "Processing: $task"
    # Simulate work
    sleep $((RANDOM % 10 + 1))
    echo "Completed: $task"
}

# Read tasks and process in parallel
while IFS= read -r task; do
    # Wait if max jobs running
    while [ $(jobs -r | wc -l) -ge $MAX_PARALLEL ]; do
        sleep 0.5
    done
    
    # Start new task
    process_task "$task" &
done < "$TASK_LIST"

# Wait for all to complete
wait
echo "All tasks completed"
```

---

## Best Practices

1. **Always handle signals** for graceful shutdown
2. **Use PID files** for daemon management
3. **Monitor resource usage** in long-running scripts
4. **Limit parallelism** to avoid overwhelming system
5. **Clean up child processes** to avoid zombies
6. **Use nohup/disown** for persistent processes
7. **Log process events** for debugging

---

## Summary

- Processes have states and can be controlled with signals
- Use background jobs (`&`) and job control (`bg`, `fg`)
- `trap` handles signals for cleanup
- `nohup` and `disown` for persistent processes
- `wait` synchronizes parallel processes
- Monitor resources with `ps`, `top`, custom scripts

---

## Practice Exercises

1. Write a process manager that can start, stop, and restart a daemon
2. Create a resource monitor that alerts on high CPU/memory usage
3. Build a parallel task processor with configurable concurrency
4. Implement a script that gracefully handles SIGTERM and SIGINT

---

**Next Chapter:** Part 7 - System Administration with Bash
