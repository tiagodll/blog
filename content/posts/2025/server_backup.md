---
title: "Backing up config files"
date: 2025-07-30T12:30:00+02:00
draft: false
tags: ['server', 'linux']
description: How do you backup your config files? you are not saving it in github right?
---

Recently I have reset instead of restart my server.

Not great, lost a lot of configuration I had to remake and without proper backups this was a waste of time.

I have been postponing this automated backup since forever, and this was the push I needed to actually do it.

So naturally as any developer, I started creating a software for it.

But as always, I underestimated the ammount of effort for it.

After half day on it, I decided it wasnt worth it, and go for something simpler.

So I came up with this:
- rclone to upload my files
- stupid simple script to take files and push it to my storage of choice but only if they have been modified in the past 24h
- use crontab to run it every night

### backup file when older than 24h
```bash
#!/bin/bash

function backup {
    local FILE="$2"
    local WEBPATH="$1"

    # Check if file exists
    if [ ! -f "$FILE" ]; then
        echo "Error: File '$FILE' does not exist"
        return 1
    fi

    # Current time in seconds since epoch
    local CURRENT_TIME
    CURRENT_TIME=$(date +%s)

    # 24 hours in seconds
    local ONE_DAY=86400

    # Get file's last modification time in seconds
    local MOD_TIME
    MOD_TIME=$(stat -c %Y "$FILE" 2>/dev/null || stat -f %m "$FILE" 2>/dev/null)

    # Check if stat command succeeded
    if [ -z "$MOD_TIME" ]; then
        echo "Error: Could not retrieve modification time for '$FILE'"
        return 1
    fi

    # Calculate time difference
    local TIME_DIFF=$((CURRENT_TIME - MOD_TIME))

    # Check if file was modified within last 24 hours
    if [ $TIME_DIFF -le $ONE_DAY ]; then
        echo "Executing: rclone copy $FILE pcloud:misc/backups/$WEBPATH"
        rclone copy "$FILE" "pcloud:misc/backups/$WEBPATH"
    else
        echo "Skipping old file: $FILE"
    fi
}

backup config /etc/caddy/Caddyfile
backup config /var/backups/backup_modified.sh

backup projectx /etc/systemd/system/projectx.service
backup projectx /var/www/projectx/projectx.db

```

### Crontab
```bash
00 01 * * * /var/backups/backup_modified.sh 2>&1 | /usr/bin/logger -t backup_projects
```

So not the perfect solution, but it works.