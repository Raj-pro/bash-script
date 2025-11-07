# Chapter 18: Date, Time, and Scheduling

## Introduction

Working with dates, times, and scheduled tasks is essential for automation. This chapter covers the `date` command, timestamp manipulation, and scheduling with cron and systemd timers.

---

## The date Command

### Basic Usage

```bash
# Current date and time
date

# Output: Fri Nov  8 14:30:45 PST 2025

# Custom format
date +"%Y-%m-%d"           # 2025-11-08
date +"%H:%M:%S"           # 14:30:45
date +"%Y-%m-%d %H:%M:%S"  # 2025-11-08 14:30:45

# Full day name
date +"%A"                 # Friday

# Short day name
date +"%a"                 # Fri

# Full month name
date +"%B"                 # November

# Short month name
date +"%b"                 # Nov
```

### Date Format Specifiers

```bash
#!/bin/bash

# Common format codes
%Y  # Year (4 digits)         - 2025
%y  # Year (2 digits)         - 25
%m  # Month (01-12)           - 11
%d  # Day (01-31)             - 08
%H  # Hour (00-23)            - 14
%M  # Minute (00-59)          - 30
%S  # Second (00-59)          - 45
%A  # Full weekday name       - Friday
%a  # Short weekday name      - Fri
%B  # Full month name         - November
%b  # Short month name        - Nov
%Z  # Time zone               - PST
%s  # Unix timestamp          - 1730000000

# Examples
echo "Today is $(date +%A), $(date +%B) $(date +%d), $(date +%Y)"
# Output: Today is Friday, November 08, 2025

echo "Current time: $(date +%I:%M:%S\ %p)"
# Output: Current time: 02:30:45 PM

echo "ISO 8601: $(date +%Y-%m-%dT%H:%M:%S%z)"
# Output: ISO 8601: 2025-11-08T14:30:45-0800
```

---

## Date Arithmetic

### Relative Dates

```bash
#!/bin/bash

# Tomorrow
date -d "tomorrow" +"%Y-%m-%d"
date -d "1 day" +"%Y-%m-%d"

# Yesterday
date -d "yesterday" +"%Y-%m-%d"
date -d "1 day ago" +"%Y-%m-%d"

# Specific days ahead/behind
date -d "7 days" +"%Y-%m-%d"        # 1 week from now
date -d "30 days ago" +"%Y-%m-%d"   # 30 days ago
date -d "2 weeks" +"%Y-%m-%d"       # 2 weeks from now
date -d "3 months" +"%Y-%m-%d"      # 3 months from now

# Next Monday
date -d "next monday" +"%Y-%m-%d"

# Last Friday
date -d "last friday" +"%Y-%m-%d"

# Specific date
date -d "2025-12-25" +"%A"          # What day is Christmas?
```

### Date Calculations

```bash
#!/bin/bash

# Days between two dates
date1="2025-11-08"
date2="2025-12-25"

timestamp1=$(date -d "$date1" +%s)
timestamp2=$(date -d "$date2" +%s)

diff_seconds=$((timestamp2 - timestamp1))
diff_days=$((diff_seconds / 86400))

echo "Days until Christmas: $diff_days"

# Age calculation
birthdate="1990-01-01"
today=$(date +%s)
birth=$(date -d "$birthdate" +%s)
age_seconds=$((today - birth))
age_years=$((age_seconds / 31557600))  # 365.25 days

echo "Age: $age_years years"

# Working days calculation (excluding weekends)
count_working_days() {
    local start_date=$1
    local end_date=$2
    local count=0
    
    local current=$(date -d "$start_date" +%s)
    local end=$(date -d "$end_date" +%s)
    
    while [ $current -le $end ]; do
        day_of_week=$(date -d "@$current" +%u)
        if [ $day_of_week -lt 6 ]; then  # Monday-Friday
            ((count++))
        fi
        current=$((current + 86400))
    done
    
    echo $count
}

working_days=$(count_working_days "2025-11-01" "2025-11-30")
echo "Working days in November: $working_days"
```

---

## Unix Timestamps

### Working with Timestamps

```bash
#!/bin/bash

# Current Unix timestamp
timestamp=$(date +%s)
echo "Timestamp: $timestamp"  # 1730000000

# Convert timestamp to readable date
date -d "@$timestamp"

# Convert date to timestamp
date -d "2025-11-08 14:30:45" +%s

# Timestamp arithmetic
now=$(date +%s)
one_hour_ago=$((now - 3600))
tomorrow=$((now + 86400))

echo "One hour ago: $(date -d "@$one_hour_ago")"
echo "Tomorrow: $(date -d "@$tomorrow")"

# Compare timestamps
if [ $timestamp1 -gt $timestamp2 ]; then
    echo "Date 1 is after Date 2"
fi
```

---

## Timestamping Files and Logs

### Create Timestamped Files

```bash
#!/bin/bash

# Timestamp in filename
timestamp=$(date +%Y%m%d_%H%M%S)
log_file="backup_${timestamp}.log"
echo "Creating: $log_file"

# Backup with timestamp
backup_dir="/backup/$(date +%Y/%m/%d)"
mkdir -p "$backup_dir"
tar -czf "$backup_dir/backup_$(date +%H%M%S).tar.gz" /data

# Rotate logs with date
log_dir="/var/log/myapp"
current_log="$log_dir/app.log"
archive_log="$log_dir/app_$(date +%Y%m%d).log"

if [ -f "$current_log" ]; then
    mv "$current_log" "$archive_log"
    touch "$current_log"
fi
```

### Logging with Timestamps

```bash
#!/bin/bash

LOG_FILE="/var/log/script.log"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

# Usage
log "INFO: Script started"
log "ERROR: Something went wrong"
log "INFO: Script completed"

# Output:
# [2025-11-08 14:30:45] INFO: Script started
# [2025-11-08 14:30:46] ERROR: Something went wrong
# [2025-11-08 14:30:47] INFO: Script completed
```

---

## Scheduling with Cron

### Cron Syntax

```
* * * * * command
│ │ │ │ │
│ │ │ │ └─── Day of week (0-7, Sunday = 0 or 7)
│ │ │ └───── Month (1-12)
│ │ └─────── Day of month (1-31)
│ └───────── Hour (0-23)
└─────────── Minute (0-59)
```

### Common Cron Examples

```bash
# Edit crontab
crontab -e

# List current crontab
crontab -l

# Remove crontab
crontab -r

# Cron examples:

# Every minute
* * * * * /path/to/script.sh

# Every hour at minute 0
0 * * * * /path/to/script.sh

# Every day at midnight
0 0 * * * /path/to/script.sh

# Every day at 2:30 AM
30 2 * * * /path/to/script.sh

# Every Monday at 9:00 AM
0 9 * * 1 /path/to/script.sh

# First day of every month at midnight
0 0 1 * * /path/to/script.sh

# Every 15 minutes
*/15 * * * * /path/to/script.sh

# Every 6 hours
0 */6 * * * /path/to/script.sh

# Weekdays at 8:00 AM
0 8 * * 1-5 /path/to/script.sh

# Multiple times
0 8,12,18 * * * /path/to/script.sh
```

### Cron with Environment Variables

```bash
# Set environment variables in crontab
SHELL=/bin/bash
PATH=/usr/local/bin:/usr/bin:/bin
MAILTO=admin@example.com

# Use variables in commands
0 2 * * * /path/to/backup.sh >> /var/log/backup.log 2>&1

# Redirect output
0 * * * * /path/to/script.sh > /dev/null 2>&1
```

### Cron Best Practices

```bash
#!/bin/bash
# cron-script.sh

# Use absolute paths
PATH=/usr/local/bin:/usr/bin:/bin

# Set up logging
LOG_FILE="/var/log/cron-script.log"

# Lock file to prevent concurrent runs
LOCK_FILE="/var/run/cron-script.lock"

# Check for lock
if [ -f "$LOCK_FILE" ]; then
    echo "Script already running" >> "$LOG_FILE"
    exit 1
fi

# Create lock
touch "$LOCK_FILE"
trap "rm -f '$LOCK_FILE'" EXIT

# Your script logic
echo "[$(date)] Script started" >> "$LOG_FILE"

# Do work...

echo "[$(date)] Script completed" >> "$LOG_FILE"
```

---

## Systemd Timers

### Create Systemd Timer

```bash
# Create service file: /etc/systemd/system/backup.service
cat > /etc/systemd/system/backup.service << 'EOF'
[Unit]
Description=Backup Service
Wants=backup.timer

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh

[Install]
WantedBy=multi-user.target
EOF

# Create timer file: /etc/systemd/system/backup.timer
cat > /etc/systemd/system/backup.timer << 'EOF'
[Unit]
Description=Backup Timer
Requires=backup.service

[Timer]
# Run daily at 2:00 AM
OnCalendar=*-*-* 02:00:00
# Run 5 minutes after boot
OnBootSec=5min
# Randomize start time by up to 10 minutes
RandomizedDelaySec=10min

[Install]
WantedBy=timers.target
EOF

# Enable and start timer
sudo systemctl daemon-reload
sudo systemctl enable backup.timer
sudo systemctl start backup.timer

# Check timer status
sudo systemctl status backup.timer
sudo systemctl list-timers
```

### Systemd Timer Examples

```bash
# Every day at midnight
OnCalendar=*-*-* 00:00:00

# Every Monday at 9:00 AM
OnCalendar=Mon *-*-* 09:00:00

# Every hour
OnCalendar=hourly

# Every 15 minutes
OnCalendar=*:0/15

# First day of month
OnCalendar=*-*-01 00:00:00

# Every 6 hours
OnCalendar=0/6:00:00

# Weekdays at 8:00 AM
OnCalendar=Mon..Fri *-*-* 08:00:00
```

---

## Practical Examples

### Daily Backup with Rotation

```bash
#!/bin/bash
# daily-backup.sh

set -euo pipefail

BACKUP_DIR="/backup"
SOURCE_DIR="/data"
RETENTION_DAYS=7
DATE=$(date +%Y%m%d)
LOG_FILE="/var/log/backup.log"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

log "Starting backup..."

# Create dated backup directory
mkdir -p "$BACKUP_DIR/$DATE"

# Perform backup
tar -czf "$BACKUP_DIR/$DATE/backup.tar.gz" "$SOURCE_DIR" 2>&1 | tee -a "$LOG_FILE"

if [ ${PIPESTATUS[0]} -eq 0 ]; then
    log "Backup completed successfully"
else
    log "ERROR: Backup failed"
    exit 1
fi

# Remove old backups
find "$BACKUP_DIR" -maxdepth 1 -type d -mtime +$RETENTION_DAYS -exec rm -rf {} \;

log "Cleanup completed"

# Crontab entry:
# 0 2 * * * /usr/local/bin/daily-backup.sh
```

### Log Rotation Script

```bash
#!/bin/bash
# rotate-logs.sh

LOG_DIR="/var/log/myapp"
MAX_SIZE=100M  # 100 MB
MAX_AGE=30     # 30 days

rotate_log() {
    local log_file=$1
    local timestamp=$(date +%Y%m%d_%H%M%S)
    
    # Check if file exists and is not empty
    if [ ! -s "$log_file" ]; then
        return
    fi
    
    # Check file size
    size=$(du -b "$log_file" | cut -f1)
    max_bytes=$((100 * 1024 * 1024))  # 100 MB in bytes
    
    if [ $size -gt $max_bytes ]; then
        echo "Rotating $log_file (size: $size bytes)"
        
        # Move and compress
        mv "$log_file" "${log_file}.${timestamp}"
        gzip "${log_file}.${timestamp}"
        
        # Create new empty log
        touch "$log_file"
        
        # Set permissions
        chmod 644 "$log_file"
    fi
}

# Rotate all log files
for log in "$LOG_DIR"/*.log; do
    rotate_log "$log"
done

# Remove old compressed logs
find "$LOG_DIR" -name "*.log.*.gz" -mtime +$MAX_AGE -delete

# Crontab entry:
# 0 0 * * * /usr/local/bin/rotate-logs.sh
```

### Weekly Report Generator

```bash
#!/bin/bash
# weekly-report.sh

REPORT_DIR="/reports"
DATE=$(date +%Y%m%d)
WEEK=$(date +%U)
YEAR=$(date +%Y)

# Get date range
start_date=$(date -d "7 days ago" +%Y-%m-%d)
end_date=$(date +%Y-%m-%d)

report_file="$REPORT_DIR/weekly_report_${YEAR}_week${WEEK}.txt"

{
    echo "Weekly Report"
    echo "============="
    echo "Period: $start_date to $end_date"
    echo
    
    echo "System Uptime:"
    uptime
    echo
    
    echo "Disk Usage:"
    df -h
    echo
    
    echo "Top 5 Memory Consumers:"
    ps aux --sort=-%mem | head -n 6
    echo
    
    echo "Log Summary:"
    grep -c "ERROR" /var/log/app.log 2>/dev/null || echo "0"
    
} > "$report_file"

# Email report
# mail -s "Weekly Report - Week $WEEK" admin@example.com < "$report_file"

echo "Report generated: $report_file"

# Crontab entry (every Sunday at 11:59 PM):
# 59 23 * * 0 /usr/local/bin/weekly-report.sh
```

### Reminder System

```bash
#!/bin/bash
# reminder.sh

REMINDER_FILE="$HOME/.reminders"

add_reminder() {
    local date=$1
    local time=$2
    local message=$3
    
    echo "$date $time|$message" >> "$REMINDER_FILE"
    echo "Reminder added for $date at $time"
}

check_reminders() {
    local now=$(date +%s)
    
    while IFS='|' read -r datetime message; do
        reminder_time=$(date -d "$datetime" +%s 2>/dev/null)
        
        if [ -n "$reminder_time" ] && [ $now -ge $reminder_time ]; then
            # Show notification
            notify-send "Reminder" "$message"
            
            # Or send email
            # echo "$message" | mail -s "Reminder" user@example.com
            
            # Remove from file
            sed -i "/$datetime/d" "$REMINDER_FILE"
        fi
    done < "$REMINDER_FILE"
}

# Usage:
# add_reminder "2025-11-08" "15:00" "Team meeting"

# Crontab entry (check every minute):
# * * * * * /usr/local/bin/reminder.sh check_reminders
```

---

## Time Zones

### Working with Time Zones

```bash
#!/bin/bash

# Current timezone
echo "Current: $(date)"

# Specific timezone
TZ="America/New_York" date
TZ="Europe/London" date
TZ="Asia/Tokyo" date

# UTC
TZ="UTC" date

# Convert between timezones
convert_timezone() {
    local datetime=$1
    local from_tz=$2
    local to_tz=$3
    
    TZ="$from_tz" date -d "$datetime"
    TZ="$to_tz" date -d "$datetime"
}

# List available timezones
timedatectl list-timezones
```

---

## Best Practices

1. **Always use absolute paths** in cron jobs
2. **Redirect output** to log files
3. **Use lock files** to prevent concurrent runs
4. **Test date calculations** with edge cases
5. **Log all cron job execution** for debugging
6. **Use ISO 8601 format** for dates: `YYYY-MM-DD`
7. **Consider timezones** for distributed systems
8. **Validate date inputs** before processing

---

## Summary

- Use `date` command for date/time operations
- Calculate date differences with timestamps
- Use format specifiers for custom date formats
- Schedule tasks with cron or systemd timers
- Implement log rotation and cleanup
- Always log with timestamps
- Handle timezones appropriately

---

## Practice Exercises

1. Create a birthday reminder system
2. Implement a backup rotation script (keep 7 daily, 4 weekly, 12 monthly)
3. Generate monthly usage reports
4. Build a countdown timer for events
5. Create a system uptime tracker

---

**Next Chapter:** Part 6 - Advanced Bash Topics
