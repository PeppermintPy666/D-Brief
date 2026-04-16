# Set Up From Scratch: Automated Daily BRL/USD & Media Analysis Briefing

## Overview

This guide documents the full process of setting up an automated daily briefing system that runs Claude Code skills on a schedule, generates a report, cleans and slices it for multi-platform distribution, and publishes across email, GitHub Pages, BlueSky, and Mastodon. The system runs on a Linux PC (Ubuntu/Debian) using cron for scheduling.

### Architecture

```
Cron (scheduled trigger)
  → Bash script (~/daily-report.sh)
    → Claude Code CLI (claude -p) — 3 sequential calls:
      Phase 1-4: Full English report generation (~3-5 min)
      Phase 5:   Portuguese translation (~1-2 min)
      Phase 6:   Cross-reference check (~30-60 sec)
    → Clean (~/bin/clean-report.py)
      Phase 7:   Strip preamble, validate structure
    → Splice (~/bin/splice-report.sh)
      Phase 8:   Extract signal line, exec brief, social teaser, full report
    → Distribute
      Phase 9a:  Pandoc → HTML email via msmtp (Proton Bridge)
      Phase 9b:  Git push to GitHub Pages (must go first — links must be live)
      Phase 9c:  Wait 45s for Pages rebuild
      Phase 9d:  Post signal line to BlueSky (curl + app password)
      Phase 9e:  Post exec brief teaser to Mastodon (curl + access token)
      Phase 9f:  Substack — manual for now
```

### Prerequisites

- Linux PC (Ubuntu/Debian-based)
- Paid Proton Mail account (required for Bridge)
- Claude Pro or Max subscription (required for Claude Code)
- BlueSky account (free, app password for API)
- Mastodon account (any instance; access token via Settings → Development)
- Internet connection (VPN-compatible — all services work behind VPN)

### Dependencies to install

- Node.js 22 via nvm (for Claude Code)
- Claude Code CLI (`npm install -g @anthropic-ai/claude-code`)
- Pandoc (markdown → HTML)
- msmtp + Proton Mail Bridge (email)
- Git + GitHub Pages (archive)
- GPG (encrypted credentials)
- jq (JSON parsing for social API responses)
- curl (social API calls)

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
# Should show: brl-usd-trader/  media-analyst/  daily-briefing/
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

## Phase 6 — Install Pandoc and jq

Pandoc converts Markdown to HTML for formatted emails and the GitHub Pages archive. jq parses JSON responses from the BlueSky and Mastodon APIs.

```bash
sudo apt install pandoc jq
```

---

## Phase 7 — Set Up Social Platform Credentials

All credentials are GPG-encrypted and stored in per-service directories with restricted permissions. This limits blast radius if any single credential leaks.

### BlueSky

1. Create an account at bsky.app (pseudonymous, VPN-friendly, no phone required)
2. Settings → App Passwords → create one called "d-brief-bot"
3. Store it:

```bash
mkdir -p ~/.config/bluesky && chmod 700 ~/.config/bluesky
echo -n "YOUR_APP_PASSWORD" | gpg --encrypt --recipient YOUR_GPG_EMAIL -o ~/.config/bluesky/app-pass.gpg
```

4. Verify:

```bash
gpg --quiet --for-your-eyes-only --no-tty --decrypt ~/.config/bluesky/app-pass.gpg
# Should print the app password
```

### Mastodon

1. Create an account on your chosen instance (mastodon.social = 500 chars, journa.host = 2000 chars, etc.)
2. Navigate to `https://YOUR-INSTANCE/settings/applications`
3. New Application → name: "d-brief-bot" → scope: only `write:statuses` → Submit
4. Click into the app → copy "Your access token" (the top value)
5. Store it:

```bash
mkdir -p ~/.config/mastodon && chmod 700 ~/.config/mastodon
echo -n "YOUR_ACCESS_TOKEN" | gpg --encrypt --recipient YOUR_GPG_EMAIL -o ~/.config/mastodon/token.gpg
```

6. Verify:

```bash
gpg --quiet --for-your-eyes-only --no-tty --decrypt ~/.config/mastodon/token.gpg
# Should print the access token
```

**Security note:** If a token is ever exposed (e.g., pasted into a chat), rotate it immediately. BlueSky: delete and recreate the app password. Mastodon: regenerate the token in Settings → Development → your app.

---

## Phase 8 — Create the Processing Scripts

These two scripts sit between report generation (Claude Code) and distribution. They clean Claude's raw output and slice it into platform-specific pieces.

### Clean script (Phase 7 of the pipeline)

```bash
mkdir -p ~/bin
nano ~/bin/clean-report.py
```

Contents:

```python
#!/usr/bin/env python3
import sys, re, pathlib

if len(sys.argv) != 2:
    sys.stderr.write("Usage: clean-report.py <raw.md>\n")
    sys.exit(1)

raw = pathlib.Path(sys.argv[1]).read_text()
original_len = len(raw)

match = re.search(r"^#\s", raw, flags=re.MULTILINE)
if not match:
    sys.stderr.write("FATAL: no '# ' heading found in input\n")
    sys.exit(1)

stripped_bytes = match.start()
cleaned = raw[match.start():]

cleaned = re.sub(r"^```(?:markdown|md)?\s*\n", "", cleaned)
cleaned = re.sub(r"\n```\s*$", "\n", cleaned)
cleaned = re.sub(r"\n{3,}", "\n\n", cleaned)

for line in cleaned.splitlines()[:5]:
    for pat in [r"^(I'll now|I now have|Let me |Now I'll|Now assembling|All data collected)",
                r"^(No prior editions found)",
                r"^(Here is (the|your) (briefing|report))"]:
        if re.match(pat, line, flags=re.IGNORECASE):
            sys.stderr.write(f"WARN: suspected leak in first 5 lines: {line[:80]}\n")

missing = [s for s in ["EXECUTIVE BRIEF", "SECTION 1", "SECTION 2", "FORECAST"] if s not in cleaned]
if missing:
    sys.stderr.write(f"WARN: missing expected sections: {missing}\n")

sys.stderr.write(f"Stripped {stripped_bytes} bytes of preamble. Original: {original_len}, Cleaned: {len(cleaned)}\n")
sys.stdout.write(cleaned)
```

```bash
chmod +x ~/bin/clean-report.py
```

### Splice script (Phase 8 of the pipeline)

```bash
nano ~/bin/splice-report.sh
```

Contents:

```bash
#!/bin/bash
set -euo pipefail

CLEAN="$1"
OUTDIR="$2"
mkdir -p "$OUTDIR"

# Signal line (BlueSky) — first heading, strip the #
SIGNAL=$(head -5 "$CLEAN" | grep '^# ' | head -1 | sed 's/^# //')
if [ ${#SIGNAL} -gt 240 ]; then
    SIGNAL="${SIGNAL:0:237}..."
fi
echo "$SIGNAL" > "$OUTDIR/signal_line.txt"

# Executive Brief (full, for email)
sed -n '/^## .*EXECUTIVE BRIEF/,/^## /{/^## /d;p}' \
    "$CLEAN" > "$OUTDIR/exec_brief.md"

# Mastodon teaser (max 400 chars, first 1-2 sentences)
sed 's/\*\*//g; s/\*//g; s/\[//g; s/\]([^)]*)//g; /^$/d; /^---$/d' \
    "$OUTDIR/exec_brief.md" \
    | tr '\n' ' ' \
    | grep -oP '^[^.]*\.[^.]*\.' \
    > "$OUTDIR/exec_brief_social.txt"

CHAR_COUNT=$(wc -c < "$OUTDIR/exec_brief_social.txt")
if [ "$CHAR_COUNT" -gt 400 ]; then
    sed 's/\*\*//g; s/\*//g; s/\[//g; s/\]([^)]*)//g; /^$/d; /^---$/d' \
        "$OUTDIR/exec_brief.md" \
        | tr '\n' ' ' \
        | grep -oP '^[^.]*\.' \
        > "$OUTDIR/exec_brief_social.txt"
fi

# Full report
cp "$CLEAN" "$OUTDIR/full_report.md"

echo "=== Splice results ==="
echo "Signal line:  $(wc -c < "$OUTDIR/signal_line.txt") chars"
echo "Exec social:  $(wc -c < "$OUTDIR/exec_brief_social.txt") chars"
echo "Full report:  $(wc -l < "$OUTDIR/full_report.md") lines"
```

```bash
chmod +x ~/bin/splice-report.sh
```

### Test both scripts

Run them against a generated report to verify they work:

```bash
# Clean
~/bin/clean-report.py ~/daily-reports/SOME-REPORT-EN.md > /tmp/cleaned.md

# Splice
~/bin/splice-report.sh /tmp/cleaned.md /tmp/splice-test/

# Inspect
cat /tmp/splice-test/signal_line.txt
head -10 /tmp/splice-test/exec_brief_social.txt
```

---

## Phase 9 — Create the Daily Report Script

The main script orchestrates all nine phases. It uses heredoc syntax (`<<'DELIMITER'`)
for all Claude prompts to avoid bash quoting issues with apostrophes.

Save as `~/daily-report.sh` and make executable with `chmod +x ~/daily-report.sh`.

**Important:** After ANY edit to this script, validate before running:

```bash
bash -n ~/daily-report.sh && echo "Clean parse" || echo "Syntax error"
```

`bash -n` parses the script without executing it. It catches unclosed quotes, bad
heredocs, and syntax errors in one second with zero risk. Run it after every edit.

The script contains:
- Configuration block (email, paths, variables)
- GPG agent setup
- Phase 1-4: Orchestrator prompt (heredoc) → `claude -p` → EN report
- Phase 5: PT translation prompt (heredoc) → `claude -p` → PT report
- Phase 6: Cross-reference prompt (heredoc) → `claude -p` → verification table
- Phase 7: `~/bin/clean-report.py` → cleaned EN report
- Phase 8: `~/bin/splice-report.sh` → platform-specific slices
- Phase 9a: Pandoc + msmtp → email
- Phase 9b: Git push → GitHub Pages archive + index rebuild
- Phase 9c-e: Social posting functions (`post_bluesky`, `post_mastodon`)
- Each social channel wrapped in independent error handling

See the actual script at `~/daily-report.sh` for the current implementation.
A backup of each major version is kept as `~/daily-report.sh.vN-description`.

---

## Phase 10 — Configure GPG Cache for Unattended Operation

The GPG agent needs to cache your passphrase long enough to cover periods between logins. This applies to all GPG-encrypted credentials: Bridge password, BlueSky app password, and Mastodon access token.

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

## Phase 11 — Set Up GitHub Pages Archive

### Create repository

```bash
mkdir -p ~/D-Brief/editions
cd ~/D-Brief
git init
git checkout -b gh-pages
```

### Add SSH key to GitHub

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub
# Copy output → github.com/settings/keys → New SSH Key
```

### Connect and push

```bash
cd ~/D-Brief
git remote add origin git@github.com:YOUR_USERNAME/D-Brief.git
git add -A
git commit -m "Initial setup"
git push -u origin gh-pages
```

### Enable GitHub Pages

Go to your repo → Settings → Pages → Source: Deploy from branch → Branch: gh-pages → Save.

Site will be live at `https://YOUR_USERNAME.github.io/D-Brief/`.

---

## Phase 12 — Set Up Cron Jobs

Open the crontab editor:

```bash
crontab -e
```

If prompted, choose `nano` as the editor. Add these lines at the bottom:

```
45 6 * * * export PATH="$HOME/.nvm/versions/node/$(ls $HOME/.nvm/versions/node | tail -1)/bin:$PATH" && $HOME/daily-report.sh >> $HOME/daily-reports/cron.log 2>&1
0 21 * * * export PATH="$HOME/.nvm/versions/node/$(ls $HOME/.nvm/versions/node | tail -1)/bin:$PATH" && $HOME/daily-report.sh >> $HOME/daily-reports/cron.log 2>&1
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
0 21 * * *   ← 9:00 PM every day
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
tail -50 ~/daily-reports/cron.log
```

### Check if Bridge is running

```bash
pgrep -a protonmail-bridge && echo "Bridge up" || echo "Bridge not running"
```

If not running, launch it:

```bash
protonmail-bridge &
```

If already running, don't re-launch — it will show "Instance already exists" and "Failed to launch" which are harmless messages meaning the existing instance is fine.

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

### Social posting failures

Check the cron log for `WARN: BlueSky post failed` or `WARN: Mastodon post failed`.

**BlueSky auth failure:** Verify app password decrypts:
```bash
gpg --quiet --for-your-eyes-only --no-tty --decrypt ~/.config/bluesky/app-pass.gpg
```
If it fails, the GPG cache expired. Refresh and retry.

**Mastodon auth failure:** Same GPG check for `~/.config/mastodon/token.gpg`.

**Token rotation:** If you suspect a credential is compromised:
- BlueSky: bsky.app → Settings → App Passwords → delete and recreate
- Mastodon: Settings → Development → your app → Regenerate token
Then re-encrypt the new credential.

### bash -n reports a syntax error

This means a quoting issue in the script. Most common cause: an unescaped apostrophe inside a single-quoted string. All Claude prompts should use heredoc syntax (`<<'DELIMITER'`) to avoid this. Check recent edits.

### Important operational notes

- **Screen lock:** Cron runs fine with the screen locked. The lock screen is only a UI layer; all background services continue running.
- **Standby/suspend:** Cron does NOT run when the machine is in standby. Missed jobs are skipped, not queued.
- **Reboot:** After a reboot, Bridge must be running and GPG cache must be warm before the next scheduled job. Log in and the autostart script handles the GPG refresh automatically.

---

## File Locations Reference

| File | Purpose |
|------|---------|
| `~/daily-report.sh` | Main report generation + distribution script |
| `~/daily-report.sh.vN-*` | Versioned backups of the script |
| `~/bin/clean-report.py` | Phase 7 — strip Claude meta-commentary |
| `~/bin/splice-report.sh` | Phase 8 — extract platform-specific slices |
| `~/daily-reports/` | Generated reports (.md, .html) + splice dirs |
| `~/daily-reports/cron.log` | Cron job output log |
| `~/daily-reports/splice-YYYY-MM-DD-HHMM/` | Spliced outputs per run |
| `~/D-Brief/` | GitHub Pages repo (gh-pages branch) |
| `~/D-Brief/editions/YYYY-MM-DD/` | Archived editions (index.html + pt.html) |
| `~/D-Brief/index.html` | Archive front page (auto-rebuilt each run) |
| `~/.claude/skills/` | Installed Claude Code skills |
| `~/.claude/settings.json` | Claude Code permissions |
| `~/.msmtprc` | msmtp email configuration |
| `~/.gnupg/gpg-agent.conf` | GPG cache duration settings |
| `~/.config/protonmail/bridge-cert.pem` | Bridge TLS certificate |
| `~/.config/protonmail/bridge-pass.gpg` | Encrypted Bridge password |
| `~/.config/protonmail/.gpg-cache-timestamp` | Tracks last GPG cache refresh |
| `~/.config/bluesky/app-pass.gpg` | Encrypted BlueSky app password |
| `~/.config/mastodon/token.gpg` | Encrypted Mastodon access token |
| `~/.config/autostart/gpg-cache.desktop` | Desktop login GPG refresh trigger |
| `~/.local/bin/refresh-gpg-cache.sh` | GPG cache refresh script |
| `~/.ssh/id_ed25519` | SSH key for GitHub push |

---

*Document updated April 16, 2026. Covers the complete setup from bare Linux system to automated multi-platform daily briefing delivery.*
