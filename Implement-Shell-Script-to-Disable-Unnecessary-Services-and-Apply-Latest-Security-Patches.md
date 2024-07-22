

```
#!/bin/bash

# Define essential services
ESSENTIAL_SERVICES=("sshd" "networking" "cron")

# Function to log messages
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1"
}

# Function to check the OS type
check_os() {
    if [ -f /etc/redhat-release ]; then
        OS="RedHat"
        log "Detected OS: RedHat"
    elif [ -f /etc/debian_version ]; then
        OS="Debian"
        log "Detected OS: Debian"
    else
        log "Unsupported OS."
        exit 1
    fi
}

# Function to disable non-essential services on Debian-based systems
disable_non_essential_services_debian() {
    log "Disabling non-essential services on Debian-based system..."
    for service in $(systemctl list-unit-files --type=service --state=enabled | grep enabled | awk '{print $1}'); do
        if [[ ! " ${ESSENTIAL_SERVICES[@]} " =~ " ${service%.service} " ]]; then
            systemctl stop "$service" 2>>/var/log/system_update.log
            systemctl disable "$service" 2>>/var/log/system_update.log
            if [ $? -ne 0 ]; then
                log "Failed to disable $service"
                return 1
            fi
        fi
    done
    return 0
}

# Function to disable non-essential services on RedHat-based systems
disable_non_essential_services_redhat() {
    log "Disabling non-essential services on RedHat-based system..."
    for service in $(systemctl list-unit-files --type=service --state=enabled | grep enabled | awk '{print $1}'); do
        if [[ ! " ${ESSENTIAL_SERVICES[@]} " =~ " ${service%.service} " ]]; then
            systemctl stop "$service" 2>>/var/log/system_update.log
            systemctl disable "$service" 2>>/var/log/system_update.log
            if [ $? -ne 0 ]; then
                log "Failed to disable $service"
                return 1
            fi
        fi
    done
    return 0
}

# Function to apply the latest security patches on Debian-based systems
apply_patches_debian() {
    log "Applying latest security patches on Debian-based system..."
    apt-get update 2>>/var/log/system_update.log
    if [ $? -ne 0 ]; then
        log "Failed to update package list"
        return 1
    fi

    apt-get upgrade -y 2>>/var/log/system_update.log
    if [ $? -ne 0 ]; then
        log "Failed to upgrade packages"
        return 1
    fi

    apt-get dist-upgrade -y 2>>/var/log/system_update.log
    if [ $? -ne 0 ]; then
        log "Failed to perform dist-upgrade"
        return 1
    fi

    apt-get autoremove -y 2>>/var/log/system_update.log
    if [ $? -ne 0 ]; then
        log "Failed to autoremove packages"
        return 1
    fi
    return 0
}

# Function to apply the latest security patches on RedHat-based systems
apply_patches_redhat() {
    log "Applying latest security patches on RedHat-based system..."
    yum update -y 2>>/var/log/system_update.log
    if [ $? -ne 0 ]; then
        log "Failed to update packages"
        return 1
    fi

    yum upgrade -y 2>>/var/log/system_update.log
    if [ $? -ne 0 ]; then
        log "Failed to upgrade packages"
        return 1
    fi
    return 0
}

# Main script execution
log "Starting system update script."

check_os

if [ "$OS" = "Debian" ]; then
    disable_non_essential_services_debian
    DISABLE_EXIT_CODE=$?
    if [ $DISABLE_EXIT_CODE -ne 0 ]; then
        log "Disabling non-essential services failed."
        exit 1
    fi

    apply_patches_debian
    PATCH_EXIT_CODE=$?
    if [ $PATCH_EXIT_CODE -ne 0 ]; then
        log "Applying patches failed."
        exit 1
    fi

elif [ "$OS" = "RedHat" ]; then
    disable_non_essential_services_redhat
    DISABLE_EXIT_CODE=$?
    if [ $DISABLE_EXIT_CODE -ne 0 ]; then
        log "Disabling non-essential services failed."
        exit 1
    fi

    apply_patches_redhat
    PATCH_EXIT_CODE=$?
    if [ $PATCH_EXIT_CODE -ne 0 ]; then
        log "Applying patches failed."
        exit 1
    fi
fi

log "System has been successfully updated and only essential services are enabled."
echo "System update script completed. Check /var/log/system_update.log for details."
```
