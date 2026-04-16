# D-Brief Project State — April 15, 2026

## What This Project Is

A bilingual (EN/PT) daily newsletter product analyzing BRL/USD exchange rate dynamics
through three integrated layers: macro-FX data, positional media analysis, and
economic theory stress-testing. Automated via Claude Code CLI, delivered via email,
archived on GitHub Pages, with Substack planned as the subscription/distribution layer.

---

## What Has Been Built

### 1. Three Claude Code Skills (fully developed)

**daily-briefing** (orchestrator) — 439 lines
- 6-phase execution sequence: Data → Narrative → Bridge → Assembly → Portuguese → Cross-reference
- Combined report template with 9 mandatory sections
- Scope calibration for different request types

**brl-usd-trader** — 461 lines + 3 reference files
- BRL/USD macro FX analysis skill
- References: data-sources-and-pitfalls.md, driver-mechanics.md, source-tier-classification.md
- Source tier system (T1-T4) with temporal freshness (A-D)
- Figure Provenance Standard (sourced / ⟨derived⟩ / estimated)
- Aggregation transparency rule
- Consensus Drift Tracker, Fiscal Monitor, Fiscal Trajectory Narrative
- COPOM/FOMC Action-Reaction analysis template

**media-analyst** — 598 lines + 2 reference files
- Multi-outlet positional media analysis skill
- References: source-registry.md (16 outlets with epistemic profiles), theory-profiles.md (3 theories)
- Framing-shift alert system (🟢 consistent / 🟡 edge-frame / 🔴 opposing)
- Theoretical Stress-Test (Monetarist, Post-Keynesian, Structuralist)
- Day at a Glance dashboards (FX + Framing)

### 2. Report Template (9 sections)

1. Signal line: `BRL BULLISH (medium) @ 4.997`
2. EXECUTIVE BRIEF
3. SECTION 1 — BRL/USD Analysis (Day at a Glance FX, 6-month histories, BCB, Fed, consensus drift, fiscal trajectory, synthesis, scenarios, risks, conditional sequencing, COPOM/FOMC action-reaction)
4. SECTION 2 — Media Narratives (Day at a Glance Framing, framing-shift summary, per-story positional readings with alerts, apparent contradictions, overall blind spot)
5. SECTION 3 — From Framing to the FX Positions (media→capital flow→FX mechanism mapping)
6. SECTION 4 — 3-Point Stress-Test (three theories scored against reality)
7. FORECAST (BCB, Fed, BRL/USD 30-day, geopolitical wildcard — all testable)
8. REVIEW — Where We Were Wrong (scores past forecasts; edition 1 reviews pre-crisis consensus)
9. SOURCES (with tier key, live-blog timestamps, Source Tier Reference Table)
10. FOR THE ECON STUDENT — Key Concepts

### 3. Automation Pipeline

**daily-report.sh** (276 lines) — runs three sequential `claude -p` calls:
- Call 1 (Phases 1-4): Full English report generation (~3-5 min)
- Call 2 (Phase 5): Portuguese translation (~1-2 min)
- Call 3 (Phase 6): Cross-reference check (~30-60 sec)
- Pandoc converts to HTML
- msmtp sends via Proton Mail Bridge
- git commit + push to GitHub Pages archive

**Cron schedule**:
- 6:45 AM daily
- 9:00 PM daily

### 4. Web Artifacts (React components)

- **briefing-webpage.jsx** — bilingual web page with USD/BRL toggle, soft editorial design (Fraunces + DM Sans), themed colors per language
- **briefing-audit-v4.jsx** — audit dashboard scoring report against 10 pain points
- **ipca-consensus-trajectory.html** — Chart.js dual-axis chart (IPCA left/red, SELIC right/blue)

### 5. Supporting Documents

- **target-market-pain-points.md** — 10 pain points with product-market fit scores
- **publishing-strategy.md** — GitHub Pages + Substack distribution plan + subscription ladder
- **migration-guide.md** — old script → new script deployment steps
- **daily-briefing-setup-guide.md** — original infrastructure setup (Bridge, msmtp, GPG, cron)

---

## Where Things Live

### On the Linux PC

| Path | What |
|------|------|
| `~/daily-report.sh` | The automation script (v5, 6-phase) |
| `~/daily-report.sh.old` | Backup of the v1 script |
| `~/daily-reports/` | Generated reports (EN.md, PT.md, .html) + cron.log |
| `~/D-Brief/` | GitHub Pages repo (gh-pages branch) |
| `~/.claude/skills/brl-usd-trader/` | BRL/USD trader skill |
| `~/.claude/skills/media-analyst/` | Media analyst skill |
| `~/.claude/skills/daily-briefing/` | Orchestrator skill |
| `~/.msmtprc` | Email config (Proton Bridge) |
| `~/.config/protonmail/` | Bridge cert + encrypted password |
| `~/.gnupg/gpg-agent.conf` | GPG cache (96h TTL) |
| `~/.claude/settings.json` | Claude Code permissions (WebFetch, WebSearch) |

### On GitHub

| URL | What |
|-----|------|
| `github.com/PeppermintPy666/D-Brief` | Source repo |
| `PeppermintPy666.github.io/D-Brief/` | Live archive site |
| Branch: `gh-pages` | Deployment branch |

### SSH

- Key: `~/.ssh/id_ed25519` (added to GitHub account settings)

---

## Current Status (as of end of this chat)

- ✅ Skills fully developed and packaged
- ✅ Report template complete (9 sections + all standards)
- ✅ Script updated to 6-phase orchestration
- ✅ Cron jobs set (6:45 AM + 9:00 PM)
- ✅ GitHub repo created, SSH key working, gh-pages branch deployed
- ✅ GitHub Pages enabled (Deploy from branch → gh-pages)
- ⚠️ Proton Bridge must be running for email to work (start with `protonmail-bridge &`)
- ⚠️ Script line 1 has a stray character — fix: first line must be exactly `#!/bin/bash`
- ⚠️ First full test run may have failed on email step — check `~/daily-reports/cron.log`
- ⏳ Skills need to be installed into `~/.claude/skills/` (may not be done yet)
- ⏳ Substack cross-posting not yet set up (manual for now)
- ⏳ Web page (React artifact) not yet deployed to GitHub Pages
- ⏳ Report-template.html for styled Pandoc output not yet created in ~/D-Brief/

---

## Key Design Decisions Made

1. **Source tier system (T1-T4)** with temporal freshness (A-D) on every citation
2. **Figure Provenance Standard**: sourced / ⟨derived: formula⟩ / estimated — inline, no footnotes
3. **Aggregation rule**: totals show components via ⟨derived⟩ notation
4. **Framing-shift alerts**: 🟢/🟡/🔴 per outlet per story, tracked across editions
5. **Day at a Glance dashboards**: FX (10 indicators) and Framing (per-outlet status)
6. **COPOM/FOMC action-reaction**: causal interaction matrix, not calendar co-occurrence
7. **Forecast section**: 4 testable predictions with diagnostic tests, scored in subsequent editions
8. **Review section**: scores past forecasts, builds cumulative accuracy over 30 editions
9. **Bilingual**: EN + PT with cross-reference check (16 data points verified)
10. **Subscription ladder**: Free (signal + brief + forecast) / Paid (full depth) / Premium (PT + archive)
