# D-Brief — Pending Items & Troubleshooting

## Immediate Fixes Needed

### 1. Fix script line 1
```bash
nano ~/daily-report.sh
```
Line 1 must be exactly `#!/bin/bash` — remove any stray character before the `#`.

### 2. Verify skills are installed
```bash
ls ~/.claude/skills/brl-usd-trader/SKILL.md
ls ~/.claude/skills/media-analyst/SKILL.md
ls ~/.claude/skills/daily-briefing/SKILL.md
```
If any are missing, install from the skill package zip:
```bash
cd ~/Downloads
unzip full-skill-package.zip
cp -r full-package/brl-usd-trader ~/.claude/skills/
cp -r full-package/media-analyst ~/.claude/skills/
mkdir -p ~/.claude/skills/daily-briefing
cp full-package/SKILL.md ~/.claude/skills/daily-briefing/SKILL.md
```

### 3. Verify Proton Bridge starts on login
Check if the autostart entry exists:
```bash
cat ~/.config/autostart/gpg-cache.desktop
```
Bridge itself needs to be running before the cron job fires. Either:
- Start manually each morning: `protonmail-bridge &`
- Or add to startup applications in your desktop environment

### 4. Test the full pipeline manually
```bash
# Make sure Bridge is running
protonmail-bridge &
# Wait 5 seconds
sleep 5
# Run the script
~/daily-report.sh
```
Check results:
```bash
# Did it generate?
ls -la ~/daily-reports/briefing-$(date +%Y-%m-%d)-*
# Did it deploy?
cd ~/D-Brief && git log --oneline -3
# Check the log
cat ~/daily-reports/cron.log
```

---

## Pending Development

### High Priority
- [ ] Create `~/D-Brief/report-template.html` — styled Pandoc template for pretty HTML output
- [ ] Deploy the React webpage artifact to GitHub Pages (currently only the Pandoc HTML deploys)
- [ ] First successful end-to-end test (generate → email → archive → verify GitHub Pages)

### Medium Priority
- [ ] Substack account setup + first manual cross-post
- [ ] Portuguese-language standalone edition testing
- [ ] Accuracy tracking system (score forecasts starting edition 2)
- [ ] Framing-shift tracking across editions (cumulative 🟡 → 🟢/🔴 data after 30 editions)

### Future
- [ ] Expand source registry (Nikkei Asia, SCMP, Folha de São Paulo)
- [ ] Substack API automation (fragile — see publishing-strategy.md)
- [ ] Custom domain for GitHub Pages
- [ ] RSS feed generation for the archive
- [ ] Monthly fiscal trajectory chart (visual, not just table)
- [ ] Landing page / pitch deck for subscribers

---

## Troubleshooting Reference

### "msmtp: Connection refused"
Bridge isn't running.
```bash
protonmail-bridge &
```

### "claude: command not found" (in cron)
Cron doesn't load nvm. The PATH export in the crontab line handles this. Verify:
```bash
crontab -l
```
Each line must start with:
```
export PATH="$HOME/.nvm/versions/node/$(ls $HOME/.nvm/versions/node | tail -1)/bin:$PATH" &&
```

### "git push failed"
SSH key issue or GitHub auth expired.
```bash
ssh -T git@github.com
```
Should say "Hi PeppermintPy666..." — if not, re-add the key at github.com/settings/keys.

### "GPG: decryption failed"
GPG cache expired (96h TTL). Refresh:
```bash
~/.local/bin/refresh-gpg-cache.sh
```
Or manually decrypt to re-cache:
```bash
gpg --quiet --decrypt ~/.config/protonmail/bridge-pass.gpg > /dev/null
```

### "TLS fingerprint mismatch" (msmtp)
Bridge regenerated its certificate. Re-extract:
```bash
openssl s_client -connect 127.0.0.1:1025 -starttls smtp </dev/null 2>/dev/null | openssl x509 > ~/.config/protonmail/bridge-cert.pem
openssl x509 -in ~/.config/protonmail/bridge-cert.pem -fingerprint -sha256 -noout
```
Update fingerprint in `~/.msmtprc`.

### Cron ran but no report generated
Check the log:
```bash
tail -50 ~/daily-reports/cron.log
```
Common causes: machine was in standby (cron doesn't run during sleep),
Bridge wasn't running, Claude Code auth expired (re-run `claude` interactively to re-auth).

### GitHub Pages shows 404
Either the push failed or Pages hasn't built yet (takes 1-2 minutes after push).
Check:
```bash
cd ~/D-Brief && git log --oneline -1
```
If the commit is there, wait a minute and refresh. If no commit, the script's deploy
step failed — check `~/daily-reports/cron.log` for git errors.
