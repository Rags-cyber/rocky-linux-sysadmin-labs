#!/bin/bash

USER_FILE="users.txt"
GROUP="staff"
LOG="/home/$USER/user_provision.log"

# Check the input file exists
if [ ! -f "$USER_FILE" ]; then
    echo "Error: $USER_FILE not found!"
    exit 1
fi

# Create the group if it does not exist
if ! getent group $GROUP > /dev/null; then
    groupadd $GROUP
    echo "Group $GROUP created" >> $LOG
fi

while IFS= read -r USERNAME; do
    USERNAME=$(echo "$USERNAME" | tr -d '\r')   # strip Windows carriage return
    [ -z "$USERNAME" ] && continue              # skip empty lines

    if id "$USERNAME" &>/dev/null; then
        echo "User $USERNAME already exists — skipping" >> $LOG
        continue
    fi

    useradd -m -G $GROUP -s /bin/bash $USERNAME
    echo "${USERNAME}:ChangeMe@123" | chpasswd
    passwd --expire $USERNAME
    echo "User $USERNAME created" >> $LOG

done < "$USER_FILE"

echo "Provisioning complete. Check $LOG for details."
