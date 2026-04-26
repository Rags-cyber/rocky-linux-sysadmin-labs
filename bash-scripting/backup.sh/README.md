#!/bin/bash

# ── Configuration ────────────────────────────────
SOURCE_DIR="/etc"
BACKUP_DIR="/home/$USER/backups"
LOG_FILE="/home/$USER/backup.log"
DATE=$(date +%Y-%m-%d_%H-%M-%S)
BACKUP_FILE="$BACKUP_DIR/backup_$DATE.tar.gz"
KEEP_DAYS=7

# ── Create backup directory if missing ───────────
mkdir -p $BACKUP_DIR
echo "===== Backup Started: $(date) =====" >> $LOG_FILE

# ── Create the compressed backup ─────────────────
tar -czf $BACKUP_FILE $SOURCE_DIR 2>> $LOG_FILE

# ── Check if backup succeeded ────────────────────
if [ $? -eq 0 ]; then
    SIZE=$(du -sh $BACKUP_FILE | cut -f1)
    echo "Backup successful: $BACKUP_FILE ($SIZE)" >> $LOG_FILE
else
    echo "Backup FAILED!" >> $LOG_FILE
    exit 1
fi

# ── Delete backups older than 7 days ─────────────
find $BACKUP_DIR -name 'backup_*.tar.gz' -mtime +$KEEP_DAYS -delete
echo "Old backups cleaned" >> $LOG_FILE
echo "===== Backup Done: $(date) =====" >> $LOG_FILE
