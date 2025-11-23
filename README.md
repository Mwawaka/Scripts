# System-level timers
They runs with root privileges automatically.

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

## Check current timezone
timedatectl

## If not UTC+3, set it (example: East Africa Time)
sudo timedatectl set-timezone Africa/Nairobi

# User-level Timers
**user-level systemd** timers in `~/.config/systemd/user/`. 

However, there's an important issue: **user timers can't run `sudo` commands** because they run with your user privileges, not root privileges.

## Configure Passwordless Sudo (If You Must Use User Timers)

If you really want to keep them in `~/.config/systemd/user/`, you need to:

1. **Create the script:**
```bash
mkdir -p ~/scripts
nano ~/scripts/system-update.sh
```

Content:
```bash
#!/bin/bash
sudo apt update && sudo apt upgrade -y && sudo apt autoclean && sudo apt autoremove -y
```

Make executable:
```bash
chmod +x ~/scripts/system-update.sh
```

2. **Configure passwordless sudo for apt commands:**
```bash
sudo visudo
```

Add this line at the end:
```
your_username ALL=(ALL) NOPASSWD: /usr/bin/apt
```

Replace `your_username` with your actual username.

3. **Your existing user service/timer files stay as they are:**

`~/.config/systemd/user/system-update.service`:
```ini
[Unit]
Description=System Update Script

[Service]
Type=oneshot
ExecStart=/home/your_username/scripts/system-update.sh
```

`~/.config/systemd/user/system-update.timer`:
```ini
[Unit]
Description=Run System Updates at 5 AM and 8 PM

[Timer]
OnCalendar=*-*-* 05,20:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

4. **Enable:**
```bash
systemctl --user daemon-reload
systemctl --user enable --now system-update.timer
```

## Which Should You Choose?

**System-level timers** (Option 1) are the standard approach for system maintenance tasks. User timers are typically for user-specific tasks like backups, syncing, etc.

Which approach would you prefer?