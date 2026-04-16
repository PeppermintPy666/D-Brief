# D-Brief — Technical Reference for New Chat Sessions

## How to Use This File

Upload this file at the start of any new Claude chat about this project.
It contains enough context for Claude to continue without re-reading the
full conversation history (which was ~50+ messages across 8+ hours).

---

## Product Summary

A daily bilingual (EN/PT) newsletter analyzing BRL/USD through three layers:
1. Macro-FX data (SELIC, Fed, DXY, carry, fiscal, commodities)
2. Positional media analysis (outlet-by-outlet framing with epistemic profiles)
3. Economic theory stress-test (Monetarist vs Post-Keynesian vs Structuralist)

Automated via Claude Code CLI. Delivered via email (Proton Bridge). Archived
on GitHub Pages. Substack planned for distribution/monetization.

---

## Skill Architecture

Three skills in `~/.claude/skills/`:

```
daily-briefing/SKILL.md          ← Orchestrator (6 phases, report template)
brl-usd-trader/SKILL.md          ← FX analysis
  └─ references/
     ├─ data-sources-and-pitfalls.md
     ├─ driver-mechanics.md
     └─ source-tier-classification.md   ← T1-T4 tier system + freshness A-D
media-analyst/SKILL.md            ← Media analysis
  └─ references/
     ├─ source-registry.md        ← 16 outlet epistemic profiles
     └─ theory-profiles.md        ← Monetarist, Post-Keynesian, Structuralist
```

## Report Structure (9 sections)

```
Signal line: BRL [BULLISH/BEARISH/MIXED] ([confidence]) @ [spot]
EXECUTIVE BRIEF
SECTION 1 — BRL/USD Analysis
  ├─ Day at a Glance — FX (10-indicator dashboard)
  ├─ 6-month histories (USD/BRL, SELIC, IPCA, fiscal)
  ├─ BCB standpoint → Fed standpoint → Divergence
  ├─ Global layer → Fiscal monitor + trajectory narrative
  ├─ Consensus drift tracker → Synthesis → Scenarios → Risks
  ├─ Conditional sequencing (What to Watch)
  └─ COPOM/FOMC Action-Reaction (when both within 7 days)
SECTION 2 — Media Narratives
  ├─ Day at a Glance — Framing (per-outlet alert dashboard)
  ├─ Today's Framing-Shift Summary (before stories)
  ├─ Per-story positional readings (🟢/🟡/🔴 alerts, full paragraphs)
  ├─ Apparent contradictions
  └─ Overall blind spot
SECTION 3 — From Framing to the FX Positions
  ├─ Outlet → capital pool → FX mechanism → BRL direction
  ├─ Framing Risk Map table
  └─ Convergence risk check
SECTION 4 — 3-Point Stress-Test
  ├─ Data inputs → What each theory predicts
  ├─ Reality check table → Scorecard (N/5 per theory)
  └─ Convergence (high confidence) → Divergence (genuine uncertainty)
FORECAST
  ├─ BCB stance (next COPOM) — with conditions + surprise scenario
  ├─ Fed stance (next FOMC) — tone variable + next actual move
  ├─ BRL/USD 30-day range — scenario table + asymmetry assessment
  └─ Geopolitical wildcard — probability + diagnostic test
REVIEW — Where We Were Wrong
  ├─ Scorecard of prior forecasts (edition 2+)
  ├─ Error analysis (why, not just what)
  └─ Cumulative accuracy (after 10+ editions)
SOURCES (with tier key + Source Tier Reference Table)
FOR THE ECON STUDENT — Key Concepts (5-8 terms + 3 readings)
```

## Key Standards (apply to every report)

### Source Tier System
- **T1** Primary issuer (BCB, Fed, IBGE, IMF, B3)
- **T2** Licensed redistributor (Trading Economics, MacroMicro, FRED, Yahoo Finance)
- **T3** Analytical estimate (Goldman, SWFI, Global SWF, Deloitte, ICI)
- **T4** Secondary report (Rio Times, CNBC, CNN, NPR)
- Format: `([Source](url) · TN-X, date)` where X = freshness (A≤7d, B=8-14d, C=15-21d, D=22+d)

### Figure Provenance
- **Sourced**: `value ([Source](url) · TN-X, date)`
- **Derived**: `value ⟨derived: formula⟩`
- **Estimated**: `~value ([Estimator](url) · T3; entity does not disclose)`
- **Aggregation rule**: totals show components: `~$2.4T ⟨derived: $1.1T + $750B + $510B⟩`

### Framing-Shift Alerts
- 🟢 Frame-consistent (no explanation needed)
- 🟡 Edge-frame (one sentence: what's unusual + why it matters)
- 🔴 Frame-opposing (full paragraph: expected frame, actual frame, what departure signals)
- Track across editions: 2 consecutive 🟡 on same topic = frame may be hardening

---

## Automation Pipeline

### Script: ~/daily-report.sh
```
Phase 1-4 → claude -p (orchestrated prompt, EN report)     ~3-5 min
Phase 5   → claude -p (Portuguese translation)              ~1-2 min
Phase 6   → claude -p (cross-reference check)               ~30-60 sec
Email     → pandoc + msmtp via Proton Bridge
Deploy    → git commit + push to gh-pages branch
```

### Cron: `crontab -e`
```
45 6 * * *  [PATH export] && ~/daily-report.sh >> ~/daily-reports/cron.log 2>&1
0 21 * * *  [PATH export] && ~/daily-report.sh >> ~/daily-reports/cron.log 2>&1
```

### Dependencies
- Node.js 22 via nvm (for Claude Code)
- Claude Code CLI (`npm install -g @anthropic-ai/claude-code`)
- Pandoc (markdown → HTML)
- msmtp + Proton Mail Bridge (email)
- Git + GitHub Pages (archive)
- GPG (encrypted Bridge password)

---

## GitHub Pages

- Repo: `github.com/PeppermintPy666/D-Brief`
- Branch: `gh-pages`
- URL: `PeppermintPy666.github.io/D-Brief/`
- SSH key: `~/.ssh/id_ed25519` (added to GitHub account)
- Archive structure: `editions/YYYY-MM-DD/index.html` (EN) + `pt.html` (PT)

---

## Monetization Plan

GitHub Pages = free archive (public). Substack = distribution + paywall.

| Tier | Content | Price |
|------|---------|-------|
| Free | Signal line + Executive Brief + Forecast | $0 |
| Paid | Full Sections 1-4 + Review + Econ Student | $X/mo |
| Premium | Portuguese edition + archive + cumulative data | $Y/mo |

Substack cross-posting is manual for now (copy from archive, paste into editor).
Substack API automation possible but fragile (undocumented, cookie-based auth).

---

## Files in This Project

### Downloadable outputs (from Claude chat)
- `Full-Briefing-April-15-2026-FINAL.md` — complete v9 English report (693 lines)
- `full-skill-package.zip` — all 3 skills + references
- `daily-report.sh` — automation script (276 lines)
- `briefing-webpage.jsx` — bilingual React web page
- `briefing-audit-v4.jsx` — audit dashboard
- `ipca-consensus-trajectory.html` — IPCA/SELIC chart
- `target-market-pain-points.md` — market analysis
- `publishing-strategy.md` — GitHub Pages + Substack plan
- `migration-guide.md` — old → new script deployment

### On the Linux PC
- `~/daily-report.sh` — live script
- `~/daily-reports/` — generated reports + logs
- `~/D-Brief/` — GitHub Pages repo
- `~/.claude/skills/` — installed skills

---

## What To Ask Claude Next

Likely next steps, phrased as prompts:

- "Help me debug the daily-report.sh — here's my cron.log output: [paste]"
- "Create a styled report-template.html for the Pandoc output in ~/D-Brief/"
- "Update the briefing-webpage.jsx to match the new 9-section structure with Forecast and Review"
- "Generate today's full briefing using the daily-briefing skill"
- "Set up Substack API integration for automated cross-posting"
- "Design a landing page for the D-Brief archive at PeppermintPy666.github.io/D-Brief/"
- "The script ran but the Portuguese translation failed — here's the error: [paste]"
- "Score yesterday's forecast against what actually happened (for the Review section)"
