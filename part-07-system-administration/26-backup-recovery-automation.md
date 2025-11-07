# Chapter 24: Backup and Recovery Automation

## Introduction

Automated backups are critical for data protection. This chapter covers creating robust backup scripts for local and remote systems.

---

## Basic Backup Concepts

### Types of Backups

- **Full Backup**: Complete copy of all data
- **Incremental**: Only changes since last backup
- **Differential**: Changes since last full backup
- **Mirror**: Exact copy (no compression/archive)

### Backup Best Practices

1. Follow the 3-2-1 rule: 3 copies, 2 different media, 1 offsite
2. Test restores regularly
3. Encrypt sensitive backups
4. Monitor backup success/failure
5. Rotate old backups
6. Document recovery procedures

---

## Simple File Backups

### Basic tar Backup

```bash
#!/bin/bash

SOURCE="/home/user/documents"
DEST="/backup"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_$DATE.tar.gz"

# Create compressed archive
tar -czf "$DEST/$BACKUP_FILE" "$SOURCE"

if [ $? -eq 0 ]; then
    echo "Backup successful: $BACKUP_FILE"
else
    echo "Backup failed!"
    exit 1
fi
```

### Incremental Backup with tar

```bash
#!/bin/bash

SOURCE="/data"
DEST="/backup"
SNAPSHOT_FILE="$DEST/snapshot.snar"
DATE=$(date +%Y%m%d_%H%M%S)

# Full backup if snapshot doesn't exist
if [ ! -f "$SNAPSHOT_FILE" ]; then
    echo "Creating full backup..."
    tar -czf "$DEST/full_$DATE.tar.gz" \
        --listed-incremental="$SNAPSHOT_FILE" \
        "$SOURCE"
else
    echo "Creating incremental backup..."
    tar -czf "$DEST/inc_$DATE.tar.gz" \
        --listed-incremental="$SNAPSHOT_FILE" \
        "$SOURCE"
fi
```

---

## Advanced Backup Script

### Full-Featured Backup Solution

```bash
#!/bin/bash

# Configuration
SOURCE_DIRS=("/etc" "/home" "/var/www")
BACKUP_ROOT="/backup"
RETENTION_DAYS=30
MAX_BACKUPS=10
LOG_FILE="/var/log/backup.log"
EMAIL="admin@example.com"

# Timestamp
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="$BACKUP_ROOT/$DATE"

# Logging function
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

# Error handling
error_exit() {
    log "ERROR: $1"
    # Send email alert
    # mail -s "Backup Failed" "$EMAIL" <<< "$1"
    exit 1
}

# Create backup directory
mkdir -p "$BACKUP_DIR" || error_exit "Cannot create backup directory"

log "Starting backup to $BACKUP_DIR"

# Backup each directory
for source in "${SOURCE_DIRS[@]}"; do
    if [ ! -d "$source" ]; then
        log "WARNING: $source does not exist, skipping"
        continue
    fi
    
    backup_name=$(echo "$source" | tr '/' '_')
    backup_file="$BACKUP_DIR/${backup_name}.tar.gz"
    
    log "Backing up $source..."
    
    if tar -czf "$backup_file" "$source" 2>> "$LOG_FILE"; then
        size=$(du -h "$backup_file" | cut -f1)
        log "SUCCESS: $source backed up ($size)"
    else
        error_exit "Failed to backup $source"
    fi
done

# Calculate total size
total_size=$(du -sh "$BACKUP_DIR" | cut -f1)
log "Backup completed. Total size: $total_size"

# Cleanup old backups
log "Cleaning up old backups (keeping last $MAX_BACKUPS)..."
ls -t "$BACKUP_ROOT" | tail -n +$((MAX_BACKUPS + 1)) | while read old_backup; do
    log "Removing old backup: $old_backup"
    rm -rf "$BACKUP_ROOT/$old_backup"
done

# Remove backups older than retention period
find "$BACKUP_ROOT" -maxdepth 1 -type d -mtime +$RETENTION_DAYS -exec rm -rf {} \;

log "Backup process completed successfully"
```

---

## Database Backups

### MySQL Backup

```bash
#!/bin/bash

DB_USER="backup_user"
DB_PASS="password"
DB_HOST="localhost"
BACKUP_DIR="/backup/mysql"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Get all databases
databases=$(mysql -u"$DB_USER" -p"$DB_PASS" -h"$DB_HOST" -e "SHOW DATABASES;" | tr -d "| " | grep -v Database)

for db in $databases; do
    # Skip system databases
    if [ "$db" != "information_schema" ] && [ "$db" != "performance_schema" ] && [ "$db" != "mysql" ]; then
        echo "Backing up database: $db"
        
        mysqldump -u"$DB_USER" -p"$DB_PASS" -h"$DB_HOST" \
            --single-transaction \
            --quick \
            --lock-tables=false \
            "$db" | gzip > "$BACKUP_DIR/${db}_$DATE.sql.gz"
    fi
done

echo "MySQL backup completed"
```

### PostgreSQL Backup

```bash
#!/bin/bash

export PGPASSWORD="password"
PG_USER="postgres"
PG_HOST="localhost"
BACKUP_DIR="/backup/postgresql"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Get all databases
databases=$(psql -U "$PG_USER" -h "$PG_HOST" -t -c "SELECT datname FROM pg_database WHERE datistemplate = false;")

for db in $databases; do
    db=$(echo $db | xargs)  # Trim whitespace
    
    echo "Backing up database: $db"
    
    pg_dump -U "$PG_USER" -h "$PG_HOST" "$db" | gzip > "$BACKUP_DIR/${db}_$DATE.sql.gz"
done

echo "PostgreSQL backup completed"
```

### MongoDB Backup

```bash
#!/bin/bash

MONGO_HOST="localhost"
MONGO_PORT="27017"
BACKUP_DIR="/backup/mongodb"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR/$DATE"

# Create backup
mongodump --host "$MONGO_HOST" --port "$MONGO_PORT" --out "$BACKUP_DIR/$DATE"

# Compress
tar -czf "$BACKUP_DIR/mongodb_$DATE.tar.gz" -C "$BACKUP_DIR" "$DATE"
rm -rf "$BACKUP_DIR/$DATE"

echo "MongoDB backup completed"
```

---

## Remote Backups

### Backup to Remote Server via rsync

```bash
#!/bin/bash

LOCAL_DIR="/data"
REMOTE_USER="backup"
REMOTE_HOST="backup-server.com"
REMOTE_DIR="/backups/$(hostname)"
LOG_FILE="/var/log/remote_backup.log"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

log "Starting remote backup..."

# Create remote directory if needed
ssh "$REMOTE_USER@$REMOTE_HOST" "mkdir -p $REMOTE_DIR"

# Sync with rsync
rsync -avz \
    --delete \
    --exclude='*.tmp' \
    --exclude='.cache' \
    --log-file="$LOG_FILE" \
    "$LOCAL_DIR/" \
    "$REMOTE_USER@$REMOTE_HOST:$REMOTE_DIR/"

if [ $? -eq 0 ]; then
    log "Remote backup completed successfully"
else
    log "ERROR: Remote backup failed"
    exit 1
fi
```

### Backup to Cloud (Example with AWS S3)

```bash
#!/bin/bash

SOURCE="/data"
S3_BUCKET="s3://my-backup-bucket"
DATE=$(date +%Y%m%d)
LOG_FILE="/var/log/s3_backup.log"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

log "Starting S3 backup..."

# Sync to S3
aws s3 sync "$SOURCE" "$S3_BUCKET/$DATE/" \
    --exclude "*.tmp" \
    --exclude ".cache/*" \
    --storage-class STANDARD_IA

if [ $? -eq 0 ]; then
    log "S3 backup completed"
    
    # Remove old backups (older than 30 days)
    cutoff_date=$(date -d "30 days ago" +%Y%m%d)
    aws s3 ls "$S3_BUCKET/" | while read -r line; do
        backup_date=$(echo $line | awk '{print $2}' | tr -d '/')
        if [[ "$backup_date" < "$cutoff_date" ]]; then
            log "Removing old backup: $backup_date"
            aws s3 rm "$S3_BUCKET/$backup_date/" --recursive
        fi
    done
else
    log "ERROR: S3 backup failed"
    exit 1
fi
```

---

## Backup Rotation

### Simple Rotation (Keep Last N Backups)

```bash
#!/bin/bash

BACKUP_DIR="/backup"
KEEP_BACKUPS=7

# Remove old backups
ls -t "$BACKUP_DIR"/backup_*.tar.gz 2>/dev/null | tail -n +$((KEEP_BACKUPS + 1)) | xargs -r rm

echo "Kept last $KEEP_BACKUPS backups"
```

### Grandfather-Father-Son Rotation

```bash
#!/bin/bash

BACKUP_ROOT="/backup"
SOURCE="/data"
DATE=$(date +%Y%m%d)
DAY=$(date +%A)
WEEK=$(date +%U)
MONTH=$(date +%m)

# Daily backup
mkdir -p "$BACKUP_ROOT/daily"
tar -czf "$BACKUP_ROOT/daily/backup_$DATE.tar.gz" "$SOURCE"

# Weekly backup (Sunday)
if [ "$DAY" = "Sunday" ]; then
    mkdir -p "$BACKUP_ROOT/weekly"
    cp "$BACKUP_ROOT/daily/backup_$DATE.tar.gz" "$BACKUP_ROOT/weekly/backup_week_$WEEK.tar.gz"
fi

# Monthly backup (1st of month)
if [ "$(date +%d)" = "01" ]; then
    mkdir -p "$BACKUP_ROOT/monthly"
    cp "$BACKUP_ROOT/daily/backup_$DATE.tar.gz" "$BACKUP_ROOT/monthly/backup_month_$MONTH.tar.gz"
fi

# Cleanup
find "$BACKUP_ROOT/daily" -type f -mtime +7 -delete
find "$BACKUP_ROOT/weekly" -type f -mtime +28 -delete
find "$BACKUP_ROOT/monthly" -type f -mtime +365 -delete
```

---

## Backup Verification

### Verify Backup Integrity

```bash
#!/bin/bash

verify_backup() {
    local backup_file=$1
    
    echo "Verifying backup: $backup_file"
    
    # Test archive integrity
    if tar -tzf "$backup_file" > /dev/null 2>&1; then
        echo "[OK] Archive is valid"
        
        # List contents
        file_count=$(tar -tzf "$backup_file" | wc -l)
        echo "[OK] Contains $file_count files"
        
        # Check size
        size=$(du -h "$backup_file" | cut -f1)
        echo "[OK] Size: $size"
        
        return 0
    else
        echo "[FAIL] Archive is corrupted!"
        return 1
    fi
}

# Verify all recent backups
for backup in /backup/backup_*.tar.gz; do
    verify_backup "$backup"
    echo "---"
done
```

---

## Restore Procedures

### Simple Restore

```bash
#!/bin/bash

restore_backup() {
    local backup_file=$1
    local restore_path=$2
    
    echo "Restoring $backup_file to $restore_path"
    
    mkdir -p "$restore_path"
    
    tar -xzf "$backup_file" -C "$restore_path"
    
    if [ $? -eq 0 ]; then
        echo "Restore completed successfully"
    else
        echo "Restore failed!"
        return 1
    fi
}

# Usage
restore_backup "/backup/backup_20250101.tar.gz" "/restore"
```

### Database Restore

```bash
#!/bin/bash

# MySQL restore
restore_mysql() {
    local backup_file=$1
    local db_name=$2
    
    echo "Restoring MySQL database: $db_name"
    
    gunzip < "$backup_file" | mysql -u root -p"$MYSQL_ROOT_PASSWORD" "$db_name"
}

# PostgreSQL restore
restore_postgresql() {
    local backup_file=$1
    local db_name=$2
    
    echo "Restoring PostgreSQL database: $db_name"
    
    gunzip < "$backup_file" | psql -U postgres "$db_name"
}
```

---

## Monitoring and Alerts

### Backup Monitoring Script

```bash
#!/bin/bash

BACKUP_DIR="/backup"
MAX_AGE_HOURS=24
ALERT_EMAIL="admin@example.com"

check_backup_status() {
    latest_backup=$(ls -t "$BACKUP_DIR"/backup_*.tar.gz 2>/dev/null | head -1)
    
    if [ -z "$latest_backup" ]; then
        echo "ERROR: No backups found!"
        # mail -s "Backup Alert: No backups found" "$ALERT_EMAIL"
        return 1
    fi
    
    # Check age
    backup_age=$(($(date +%s) - $(stat -c %Y "$latest_backup")))
    age_hours=$((backup_age / 3600))
    
    if [ $age_hours -gt $MAX_AGE_HOURS ]; then
        echo "ERROR: Latest backup is $age_hours hours old"
        # mail -s "Backup Alert: Backup too old" "$ALERT_EMAIL"
        return 1
    fi
    
    echo "OK: Latest backup is $age_hours hours old"
    return 0
}

check_backup_status
```

---

## Best Practices

1. **Automate backups** with cron
2. **Test restores** regularly
3. **Monitor backup success**
4. **Encrypt sensitive data**
5. **Store backups offsite**
6. **Document procedures**
7. **Rotate backups** to save space

---

## Summary

- Create automated backup scripts with tar/rsync
- Implement proper rotation strategies
- Backup databases separately
- Verify backup integrity
- Store backups remotely/offsite
- Monitor and alert on failures

---

**Next Chapter:** [25 - Security and Permissions](25-security-permissions.md)
