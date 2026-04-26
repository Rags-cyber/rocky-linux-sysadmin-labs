#!/bin/bash

SERVICES=("sshd" "httpd" "named" "dhcpd" "vsftpd" "postfix" "dovecot")
LOG="/home/$USER/service_report.log"
FAILED=0

check_service() {
    local SERVICE=$1
    if systemctl is-active --quiet $SERVICE; then
        echo "[OK]   $SERVICE"
    else
        echo "[FAIL] $SERVICE"
        FAILED=$((FAILED + 1))
    fi
}

echo "============================" | tee $LOG
echo " Service Report: $(date) "   | tee -a $LOG
echo "============================" | tee -a $LOG

for SERVICE in "${SERVICES[@]}"; do
    check_service $SERVICE | tee -a $LOG
done

echo "----------------------------" | tee -a $LOG
echo "Failed services: $FAILED"    | tee -a $LOG
