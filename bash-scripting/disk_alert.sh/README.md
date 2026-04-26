#!/bin/bash

THRESHOLD=80
LOG="/home/$USER/disk_alert.log"

df -H | grep -vE '^Filesystem|tmpfs|cdrom' | while read -r LINE; do
    USAGE=$(echo $LINE | awk '{print $5}' | sed 's/%//')
    MOUNT=$(echo $LINE | awk '{print $6}')

    if [ "$USAGE" -ge "$THRESHOLD" ]; then
        echo "$(date): WARNING — $MOUNT at ${USAGE}%" | tee -a $LOG
    else
        echo "$(date): OK — $MOUNT at ${USAGE}%" >> $LOG
    fi
done
