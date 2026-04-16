# Set Up From Scratch: Automated Daily BRL/USD & Media Analysis Briefing

## Overview

This guide documents the full process of setting up an automated daily briefing system that runs Claude Code skills on a schedule, generates an HTML-formatted report, and emails it via Proton Mail Bridge. The system runs on a Linux PC (Ubuntu/Debian) using cron for scheduling.

### Architecture

```
Cron (scheduled trigger)
  → Bash script
    → Claude Code CLI (`claude -p`) runs two skills:
      1. brl-usd-trader (BRL/USD macro FX analysis)
      2. media-analyst (multi-outlet news framing analysis)
    → Pandoc converts combined Markdown to HTML
    → msmtp sends HTML email through Proton Mail Bridge
      → Bridge encrypts and delivers via Proton's servers
```

### Prerequisites

- Linux PC (Ubuntu/Debian-based)
- Paid Proton Mail account (required for Bridge)
- Claude Pro or Max subscription (required for Claude Code)
- Internet connection

---

## Phase 1 — Install Node.js via nvm

Claude Code requires Node.js 18+. We use nvm (Node Version Manager) to install and manage Node.js versions independently of the system package manager.

### Install nvm

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
```

What this does: `curl` downloads the install script from GitHub, the pipe (`|`) sends it to `bash` which executes it. The script installs nvm into `~/.nvm/` and adds initialization lines to your `~/.bashrc`.

**Security note:** You can inspect the script before running it by replacing `| bash` with `| less`. Press `q` to exit the viewer.

Reload your shell so nvm becomes available:

```bash
source ~/.bashrc
```

### Install Node.js 22

```bash
nvm install 22
```

Verify:

```bash
node --version
# Should return v22.x.x
```

---

## Phase 2 — Install Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

No `sudo` needed — nvm installs Node in your home directory, so global packages don't require root.

### First launch and authentication

```bash
claude
```

Follow the prompts to authenticate with your Anthropic account. Exit when done with `/exit` or `Ctrl+C`.

---

## Phase 3 — Install Claude Code Skills

### Export skills from Claude.ai

In a Claude.ai conversation, ask Claude to package your skills. It will produce `.skill` files for download.

### Install skills locally

```bash
cd ~/Downloads
claude install-skill brl-usd-trader.skill
claude install-skill media-analyst.skill
```

This unpacks each skill into `~/.claude/skills/`, where Claude Code loads them automatically.

### Verify installation

```bash
ls ~/.claude/skills/
# Should show: brl-usd-trader/  media-analyst/
```

### Grant web access permissions

Claude Code needs pre-approved permission for web search and fetch since the script runs non-interactively. Create or edit the settings file:

```bash
nano ~/.claude/settings.json
```

Contents:

```json
{
  "permissions": {
    "allow": [
      "WebFetch",
      "WebSearch"
    ]
  }
}
```

Save with `Ctrl+O`, exit with `Ctrl+X`.

---

## Phase 4 — Install and Configure Proton Mail Bridge

### Install Bridge

Download the latest `.deb` from the GitHub releases page (check https://github.com/ProtonMail/proton-bridge/releases for the current version):

```bash
wget https://github.com/ProtonMail/proton-bridge/releases/download/v3.23.1/protonmail-bridge_3.23.1-1_amd64.deb
sudo dpkg -i protonmail-bridge_3.23.1-1_amd64.deb
sudo apt install -f
```

The last command resolves any missing dependencies.

### Prerequisite: keyring

Bridge requires a secret-service password manager:

```bash
apt list --installed 2>/dev/null | grep -i keyring
```

If nothing shows:

```bash
sudo apt install gnome-keyring
```

### Launch and configure

```bash
protonmail-bridge
```

In the GUI:
1. Sign in with your Proton Mail credentials
2. Click your account name
3. Note the SMTP settings:
   - Server: `127.0.0.1`
   - Port: `1025` (may vary)
   - Username: your Proton email address
   - Password: a Bridge-generated password (NOT your regular password)

Save the Bridge password — you'll need it for the next steps.

---

## Phase 5 — Install and Configure msmtp

msmtp is a lightweight command-line SMTP client.

```bash
sudo apt install msmtp msmtp-mta
```

### Extract Bridge's TLS certificate

Since Bridge uses a self-signed certificate, we pin its fingerprint for secure local communication:

```bash
mkdir -p ~/.config/protonmail
openssl s_client -connect 127.0.0.1:1025 -starttls smtp </dev/null 2>/dev/null | openssl x509 > ~/.config/protonmail/bridge-cert.pem
```

Get the certificate fingerprint:

```bash
openssl x509 -in ~/.config/protonmail/bridge-cert.pem -fingerprint -sha256 -noout
```

Copy the fingerprint value (the `AA:BB:CC:DD:...` string after the `=`).

### Encrypt the Bridge password with GPG

This avoids storing the password in plaintext.

Generate a GPG key if you don't have one:

```bash
gpg --gen-key
```

Follow the prompts (name, email, passphrase).

Encrypt the Bridge password (use `echo -n` to avoid a trailing newline):

```bash
echo -n "YOUR_BRIDGE_PASSWORD" | gpg --encrypt --recipient YOUR_GPG_EMAIL -o ~/.config/protonmail/bridge-pass.gpg
```

Verify decryption works:

```bash
gpg --quiet --for-your-eyes-only --no-tty --decrypt ~/.config/protonmail/bridge-pass.gpg
# Should print your Bridge password
```

### Configure msmtp

```bash
nano ~/.msmtprc
```

Contents (replace the fingerprint and email with your actual values):

```
defaults
auth           on
tls            on
tls_starttls   on
tls_fingerprint  AA:BB:CC:DD:EE:FF:00:11:22:33:44:55:66:77:88:99:AA:BB:CC:DD:EE:FF:00:11:22:33:44:55:66:77:88:99

account        protonmail
host           127.0.0.1
port           1025
from           YOUR_EMAIL@pm.me
user           YOUR_EMAIL@pm.me
passwordeval   gpg --quiet --for-your-eyes-only --no-tty --decrypt ~/.config/protonmail/bridge-pass.gpg

account default : protonmail
```

Lock down permissions:

```bash
chmod 600 ~/.msmtprc
```

### Test the email chain

```bash
printf "Subject: Test\n\nTest from Bridge" | msmtp YOUR_EMAIL@pm.me
```

Check your inbox — if you receive the email, the chain is working.

---

## Phase 6 — Install Pandoc

Pandoc converts Markdown to HTML for formatted emails.

```bash
sudo apt install pandoc
```

---

## Phase 7 — Create the Daily Report Script

Save the following as `~/daily-report.sh`:

```bash
#!/bin/bash
# Daily BRL/USD + Media Analysis Report
# Runs two Claude Code skills, converts to HTML, and emails the formatted report

set -euo pipefail

# --- Configuration ---
EMAIL="YOUR_EMAIL@pm.me"
REPORT_DIR="$HOME/daily-reports"
DATE=$(date +%Y-%m-%d)
TIME=$(date +%H%M)
REPORT_FILE="$REPORT_DIR/report-$DATE-$TIME.md"
HTML_FILE="$REPORT_DIR/report-$DATE-$TIME.html"

# --- GPG agent setup (needed for non-interactive password decryption) ---
export GPG_TTY=$(tty 2>/dev/null || echo "/dev/null")
gpg-connect-agent updatestartuptty /bye >/dev/null 2>&1 || true

# Ensure report directory exists
mkdir -p "$REPORT_DIR"

# --- Generate Reports ---
echo "[$(date)] Starting daily report generation..."

# BRL/USD Trading Analysis
echo "[$(date)] Running BRL/USD analysis..."
BRL_REPORT=$(claude -p "Using the brl-usd-trader skill: analyze BRL/USD exchange rate outlook for today. Cover the SELIC vs Fed Funds spread, DXY impact on the real, BCB vs Fed standpoints, capital flows between Brazil and the US, and flag risks that could invalidate the thesis." 2>/dev/null || echo "Error: BRL/USD analysis failed to generate.")

# Media Analysis
echo "[$(date)] Running media analysis..."
MEDIA_REPORT=$(claude -p "Using the media-analyst skill: give me a comprehensive press review and media analysis of today's most significant global events. Compare how different outlets frame the key stories and what each outlet's perspective reveals or obscures." 2>/dev/null || echo "Error: Media analysis failed to generate.")

# --- Combine into Markdown ---
cat > "$REPORT_FILE" << EOF
# Daily Briefing — $DATE $(date +%H:%M)

---

## BRL/USD Trading Analysis

$BRL_REPORT

---

## Media Analysis & News Digest

$MEDIA_REPORT

---

*Generated automatically at $(date "+%H:%M %Z")*
EOF

echo "[$(date)] Report saved to $REPORT_FILE"

# --- Convert to HTML ---
echo "[$(date)] Converting to HTML..."
pandoc "$REPORT_FILE" -f markdown -t html --standalone \
    --metadata title="Daily Briefing — $DATE $(date +%H:%M)" \
    --css="" \
    -V margin-top=20 \
    -o "$HTML_FILE"

# --- Email as HTML ---
echo "[$(date)] Sending email..."

{
    printf "Subject: Daily Briefing — %s %s\n" "$DATE" "$(date +%H:%M)"
    printf "Content-Type: text/html; charset=UTF-8\n"
    printf "MIME-Version: 1.0\n"
    printf "\n"
    cat "$HTML_FILE"
} | msmtp "$EMAIL"

echo "[$(date)] Done. Report emailed to $EMAIL"
```

Make it executable:

```bash
chmod +x ~/daily-report.sh
```

### Test manually

```bash
~/daily-report.sh
```

This takes a few minutes. Check your inbox for the formatted HTML report.

---

## Phase 8 — Configure GPG Cache for Unattended Operation

The GPG agent needs to cache your passphrase long enough to cover periods between logins.

### Set cache duration

```bash
nano ~/.gnupg/gpg-agent.conf
```

Contents:

```
default-cache-ttl 345600
max-cache-ttl 345600
```

This caches for 96 hours (4 days). Reload:

```bash
gpg-connect-agent reloadagent /bye
```

### Create automatic refresh on login

This script prompts for your GPG passphrase only when the cache has less than 24 hours remaining:

```bash
mkdir -p ~/.local/bin
nano ~/.local/bin/refresh-gpg-cache.sh
```

Contents:

```bash
#!/bin/bash
# Refresh GPG cache only when less than 24h remains
# Uses a timestamp file to track last refresh

TIMESTAMP_FILE="$HOME/.config/protonmail/.gpg-cache-timestamp"
BUFFER=86400        # 24 hours in seconds
MAX_TTL=345600      # 96 hours — must match gpg-agent.conf

mkdir -p "$(dirname "$TIMESTAMP_FILE")"

REFRESH_THRESHOLD=$((MAX_TTL - BUFFER))  # 72 hours

should_refresh() {
    # No timestamp file means we've never refreshed
    [ ! -f "$TIMESTAMP_FILE" ] && return 0

    LAST_REFRESH=$(stat -c %Y "$TIMESTAMP_FILE")
    NOW=$(date +%s)
    AGE=$(( NOW - LAST_REFRESH ))

    # Refresh if cache is within 24h of expiring
    [ "$AGE" -ge "$REFRESH_THRESHOLD" ] && return 0

    return 1
}

if should_refresh; then
    echo "GPG cache expires within 24h — refreshing..."
    if gpg --quiet --decrypt ~/.config/protonmail/bridge-pass.gpg >/dev/null; then
        touch "$TIMESTAMP_FILE"
        echo "GPG cache refreshed."
    else
        echo "GPG cache refresh failed."
    fi
fi
```

Make it executable:

```bash
chmod +x ~/.local/bin/refresh-gpg-cache.sh
```

### Hook into login

**Graphical login (desktop session start):**

```bash
mkdir -p ~/.config/autostart
nano ~/.config/autostart/gpg-cache.desktop
```

Contents:

```
[Desktop Entry]
Type=Application
Name=GPG Cache Refresh
Exec=/home/YOUR_USERNAME/.local/bin/refresh-gpg-cache.sh
Hidden=false
X-GNOME-Autostart-enabled=true
```

**Terminal login:**

```bash
echo '~/.local/bin/refresh-gpg-cache.sh' >> ~/.profile
```

---

## Phase 9 — Set Up Cron Jobs

Open the crontab editor:

```bash
crontab -e
```

If prompted, choose `nano` as the editor. Add these lines at the bottom:

```
45 6 * * * export PATH="$HOME/.nvm/versions/node/$(ls $HOME/.nvm/versions/node | tail -1)/bin:$PATH" && $HOME/daily-report.sh >> $HOME/daily-reports/cron.log 2>&1
45 18 * * * export PATH="$HOME/.nvm/versions/node/$(ls $HOME/.nvm/versions/node | tail -1)/bin:$PATH" && $HOME/daily-report.sh >> $HOME/daily-reports/cron.log 2>&1
```

### Cron syntax reference

```
┌───────── minute (0–59)
│ ┌─────── hour (0–23, 24h format)
│ │ ┌───── day of month (1–31)
│ │ │ ┌─── month (1–12)
│ │ │ │ ┌─ day of week (0–7, 0 and 7 are Sunday)
│ │ │ │ │
45 6 * * *   ← 6:45 AM every day
45 18 * * *  ← 6:45 PM every day
```

The `export PATH=...` segment is necessary because cron runs in a minimal environment that doesn't load nvm. This ensures cron can find the `claude` command.

`>> $HOME/daily-reports/cron.log 2>&1` appends all output and errors to a log file for troubleshooting.

### Verify

```bash
crontab -l
```

---

## Troubleshooting

### Check if cron ran

```bash
cat ~/daily-reports/cron.log
```

### Check if Bridge is running

```bash
ps aux | grep bridge
```

If not running, launch it:

```bash
protonmail-bridge &
```

### GPG passphrase prompt during cron

If the cron log shows GPG errors, the cache has expired. Log in to your machine and run:

```bash
~/.local/bin/refresh-gpg-cache.sh
```

### Claude Code permission errors

If reports contain "couldn't get permission for websearch," verify your settings:

```bash
cat ~/.claude/settings.json
```

Should contain `"WebFetch"` and `"WebSearch"` in the allow list.

### Email not sending (TLS error)

Bridge regenerates its certificate occasionally. Re-extract the fingerprint:

```bash
openssl s_client -connect 127.0.0.1:1025 -starttls smtp </dev/null 2>/dev/null | openssl x509 > ~/.config/protonmail/bridge-cert.pem
openssl x509 -in ~/.config/protonmail/bridge-cert.pem -fingerprint -sha256 -noout
```

Update the fingerprint in `~/.msmtprc`.

### Important operational notes

- **Screen lock:** Cron runs fine with the screen locked. The lock screen is only a UI layer; all background services continue running.
- **Standby/suspend:** Cron does NOT run when the machine is in standby. Missed jobs are skipped, not queued.
- **Reboot:** After a reboot, Bridge must be running and GPG cache must be warm before the next scheduled job. Log in and the autostart script handles the GPG refresh automatically.

---

## File Locations Reference

| File | Purpose |
|------|---------|
| `~/daily-report.sh` | Main report generation script |
| `~/daily-reports/` | Generated reports (`.md` and `.html`) |
| `~/daily-reports/cron.log` | Cron job output log |
| `~/.claude/skills/` | Installed Claude Code skills |
| `~/.claude/settings.json` | Claude Code permissions |
| `~/.msmtprc` | msmtp email configuration |
| `~/.gnupg/gpg-agent.conf` | GPG cache duration settings |
| `~/.config/protonmail/bridge-cert.pem` | Bridge TLS certificate |
| `~/.config/protonmail/bridge-pass.gpg` | Encrypted Bridge password |
| `~/.config/protonmail/.gpg-cache-timestamp` | Tracks last GPG cache refresh |
| `~/.config/autostart/gpg-cache.desktop` | Desktop login GPG refresh trigger |
| `~/.local/bin/refresh-gpg-cache.sh` | GPG cache refresh script |

---

*Document generated April 15, 2026. Covers the complete setup from bare Linux system to automated daily briefing delivery.*
