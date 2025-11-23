## Step 1: Create the Update Script

First, create the script file:

```bash
sudo nano /usr/local/bin/system-update.sh
```

Add this content:

```bash
#!/bin/bash
apt update && apt upgrade -y && apt autoclean && apt autoremove -y
```

Make it executable:

```bash
sudo chmod +x /usr/local/bin/system-update.sh
```

**Note:** No need for `sudo` in the script or `exit` at the end - the service will handle that.

## Step 2: Create the Service File

Create the service file (this needs to be a **system** service, not user, since it requires root):

```bash
sudo nano /etc/systemd/system/system-update.service
```

Content:

```ini
[Unit]
Description=System Update Script
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/system-update.sh
```

## Step 3: Create the Timer File

```bash
sudo nano /etc/systemd/system/system-update.timer
```

Content:

```ini
[Unit]
Description=Run System Updates Daily

[Timer]
OnCalendar=daily
OnCalendar=*-*-* 03:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

**Change `03:00:00`** to your preferred time (24-hour format).

## Step 4: Enable and Start

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now system-update.timer
```

## Verify It's Working

```bash
# Check timer status
sudo systemctl status system-update.timer

# List all timers and see when it will run next
systemctl list-timers

# Check service logs after it runs
sudo journalctl -u system-update.service
```

## OnCalendar Syntax Examples

If you want different schedules:

```ini
OnCalendar=*-*-* 03:00:00           # Daily at 3 AM
OnCalendar=Mon *-*-* 03:00:00       # Every Monday at 3 AM
OnCalendar=*-*-01 03:00:00          # First day of every month
OnCalendar=Mon,Wed,Fri 03:00:00     # Mon, Wed, Fri at 3 AM
```

The `Persistent=true` option ensures that if the system was off during the scheduled time, the update will run once the system boots up.