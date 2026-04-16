---
name: daily-briefing
description: >
  Master orchestration skill for producing the full daily BRL/USD + Geopolitical briefing.
  Use this skill whenever the user asks for a "full report", "daily briefing", "comprehensive
  analysis", "in-depth report", or any request that combines economic/FX analysis with
  geopolitical and media coverage. This skill sequences the brl-usd-trader and media-analyst
  skills in the correct order to minimize errors and maximize coherence. Also trigger for
  "April [date] report", "today's briefing", "full analysis", or "geopolitical and economic report".
  This is the entry point — it reads the other skills and orchestrates them.
---

# Daily Briefing — Master Orchestration

This skill produces the combined BRL/USD + Geopolitical + Media Analysis + Theoretical
Stress-Test report. It sequences two sub-skills in the correct order and merges their
output into a single coherent document.

## Why Order Matters

Data must be verified before narratives are built on it. If media analysis runs first,
it may cite numbers from news outlets that differ from primary sources. Getting clean
data first prevents contamination.

**The sequence is non-negotiable:**

```
Phase 1: DATA (brl-usd-trader)
   └─ Fetch all indicators from primary sources
   └─ Cross-verify every number
   └─ Classify drivers
   └─ Produce BCB vs Fed synthesis with verified data
   └─ Output: Clean data set + FX analysis

Phase 2: NARRATIVE (media-analyst)
   └─ Use Phase 1's verified data as ground truth
   └─ Search outlets for coverage of active stories
   └─ Map positional readings and framing divergences
   └─ Identify blind spots and apparent contradictions
   └─ Output: Media framing map

Phase 3: BRIDGE (media-analyst → Framing-to-FX)
   └─ Map each outlet's frame to a capital pool, FX mechanism, BRL direction
   └─ Build the Framing Risk Map table
   └─ Check for convergence risk
   └─ Output: Framing Risk Map

Phase 4: ASSEMBLY
   └─ Apply 3-point stress-test (Monetarist, Post-Keynesian, Structuralist)
   └─ Build Forecast section (4 testable predictions)
   └─ Build Review section (score prior forecasts or baseline consensus)
   └─ Write Executive Brief LAST (summarizes all sections)
   └─ Merge all phases into the Combined Report Template below
   └─ Output: Complete EN report (.md)

Phase 5: PORTUGUESE TRANSLATION
   └─ Translate the complete report into Brazilian Portuguese
   └─ Apply locale formatting, Valor Econômico register
   └─ Output: Complete PT report (.md)

Phase 6: CROSS-REFERENCE CHECK
   └─ Compare every numerical data point EN ↔ PT
   └─ Flag substantive discrepancies (not locale formatting)
   └─ Output: Verification table appended to EN report

Phase 7: CLEAN MACHINE SPEAK (~/bin/clean-report.py)
   └─ Strip any preamble or process narration before the first # heading
   └─ Remove stray code fences, collapse excess whitespace
   └─ Validate structural markers (EXECUTIVE BRIEF, SECTION 1, etc.)
   └─ Fail loud on placeholder text or missing sections
   └─ Output: Cleaned EN report (.md)

Phase 8: SPLICE (~/bin/splice-report.sh)
   └─ Extract signal line → signal_line.txt (BlueSky, ≤240 chars)
   └─ Extract Executive Brief → exec_brief.md (email, full HTML)
   └─ Build social teaser → exec_brief_social.txt (Mastodon, ≤400 chars)
   └─ Copy full report → full_report.md (website, Substack)
   └─ Output: Platform-specific slices in splice directory

Phase 9: DISTRIBUTE
   └─ 9a: Pandoc → HTML email of exec brief via msmtp (Proton Bridge)
   └─ 9b: Git push full report to gh-pages (GitHub Pages — must go first)
   └─ 9c: Wait 45s for Pages rebuild
   └─ 9d: Post signal_line.txt to BlueSky (curl + app password)
   └─ 9e: Post exec_brief_social.txt to Mastodon (curl + access token)
   └─ 9f: Substack — manual for now (copy from archive, paste into editor)
   └─ Each channel fails independently — one outage doesn't block the others
```

---

## Critical Output Rules (Phases 1–4)

Claude's output MUST comply with these rules. They prevent machine-speak from
leaking into the report and ensure the clean/splice pipeline works correctly.

- The FIRST character of output must be "#" (the signal-line heading).
- Do NOT include ANY preamble, process narration, or status updates.
- FORBIDDEN PHRASES — output must NOT contain any of these, in any form:
  * "Now assembling..." / "Let me assemble..." / "Now I'll write..."
  * "I have/I now have the data..." / "I've gathered..." / "I've collected..."
  * "All data collected..." / "Data gathering complete..."
  * "Let me now..." / "I'll now..." / "Moving on to..."
  * "Here is the briefing..." / "Below is the report..."
  * "No prior editions found..." (this is a process observation, not report content)
  * Any sentence that describes YOUR actions rather than the REPORT's content.
- Transitioning from research to writing is INTERNAL. Do not narrate it.
- Do NOT wrap the report in triple-backtick code fences.
- Do NOT add commentary after the last section.
- The report simply begins. There is no runway before it.

These rules exist because Phase 7 (clean) strips everything before the first
"# " heading. If the report starts correctly, the cleaner has nothing to do.
If it doesn't, the cleaner catches it — but prevention is better than cleanup.

---

## Before You Begin

1. Read `brl-usd-trader/SKILL.md` — this gives you the data workflow, verification
   protocol, source hierarchy, and output template.
2. Read `brl-usd-trader/references/data-sources-and-pitfalls.md` — known error patterns.
3. Read `media-analyst/SKILL.md` — this gives you the media analysis methodology,
   source registry, report templates, and the Theoretical Stress-Test section.
4. Read `media-analyst/references/source-registry.md` — epistemic profiles for all outlets.
5. Read `media-analyst/references/theory-profiles.md` — the three economic theories.

Read `brl-usd-trader/references/driver-mechanics.md` only if you need to explain a
specific transmission mechanism in detail.

---

## Combined Report Template

The final output has this structure. Every section is mandatory unless marked optional.

---

# Full Briefing — [Date]

## SIGNAL LINE (always the absolute first line — no preamble, no header before it)

> **[BRL BULLISH/BEARISH/MIXED] ([confidence]) @ [spot rate]** — [dominant force] — key risk: [risk] — next: [catalyst + date]

One line. No clickbait. A PM glancing at their phone at 6am gets the entire thesis.
This line also becomes the BlueSky post via the Phase 8 splicer.

---

## EXECUTIVE BRIEF (under 400 words for combined report)

This section serves triple duty: (1) the email body, (2) the Mastodon teaser
(first 1-2 sentences extracted by splicer), and (3) the in-report summary.
Write the first two sentences to work as a standalone social post — they should
convey the BRL thesis and dominant risk without context from the rest of the brief.

### Market Snapshot
- BRL/USD: [spot] ([source, date, hyperlinked]) — [context]
- Lean: [direction] — Confidence: [level]
- Dominant force: [what's driving BRL right now, 1 sentence]
- Rate differential: [SELIC minus Fed] bps
- Key risk: [single biggest threat to thesis]
- Next catalyst: [event + date]

### Geopolitical Snapshot
[For each active story, 1-2 sentences:]
- **[Story 1]**: [summary]. Dominant media frame: [outlet]. Key divergence: [what].
- **[Story 2]**: [summary]. Missing perspective: [what].

### Theory Check
- Best-fit theory today: [which of the three] — [1 sentence why]
- Where theories converge: [highest-confidence signal]
- Where theories diverge: [genuine uncertainty]

---

## SECTION 1 — BRL/USD Analysis

### Day at a Glance — FX (opens Section 1)
[A 30-day rolling dashboard in compact form. For each indicator, show:
current value, direction (↑/↓/→), 30-day change, and the key driver.
This gives the depth reader immediate context before the detailed analysis.

| Indicator | Current | 30d ago | Δ | Driver |
e.g., USD/BRL | 4.997 | 5.22 | −4.3% | DXY slide + commodity windfall

Include: USD/BRL, SELIC, IPCA (trailing + Focus), DXY, VIX, CDS, primary balance,
oil, iron ore. This table is the "weather report" — the reader knows the climate
before reading the forecast.]

[Full output from brl-usd-trader, following its template exactly:
BCB Standpoint → Fed Standpoint → Divergence → Global Layer →
Consensus Drift Tracker (with trajectory narrative + graph) → Fiscal Monitor →
Fiscal Trajectory → Political Calendar → Synthesis → Scenario Framework →
Key Risks → What to Watch (Conditional Sequencing)]

### COPOM / FOMC Action-Reaction Analysis (closes Section 1)
[Mandatory when COPOM and FOMC are scheduled within 7 days of each other.
This is not a calendar co-occurrence note — it is a causal interaction map.

Structure:
1. **Who moves first?** Identify which meeting concludes first (typically FOMC
   releases statement at 2pm ET; COPOM at ~6pm BRT).
2. **What does the first mover's decision change for the second?**
   Map the conditional chain: if FOMC holds + hawkish statement → COPOM calculus
   shifts toward hold. If FOMC holds + dovish → COPOM has cover for 50bps.
3. **Spread mechanics.** Show how each combination of decisions affects the
   rate differential: current (1,112bps) → if both hold → if FOMC holds + BCB cuts →
   if FOMC surprises + BCB cuts.
4. **Market positioning ahead of the pair.** What are traders likely pricing?
   Where is the surprise risk?
5. **Historical precedent.** Has this co-occurrence happened before? What was
   the BRL reaction?

This section replaces the flat "COPOM and FOMC are the same week" note with
a genuine interaction analysis.]

---

## SECTION 2 — Media Narratives

### Day at a Glance — Framing (opens Section 2)
[A 30-day rolling framing dashboard. For each major outlet tracked, show:
current alert status (🟢/🟡/🔴), how many days at that status, and
whether the frame has shifted in the past 30 days.

| Outlet | Today | Streak | 30d shift? |
e.g., Bloomberg | 🟡 | Day 2 | Was 🟢 for 28 days → edge-frame on "China proxy"

This is the "framing weather report" — the reader sees the epistemic landscape
before diving into per-story detail. Over 30 editions, patterns emerge:
which outlets are stable (always 🟢), which are volatile (frequent 🟡),
and which have permanently shifted (🟡 → 🟢 at a new position).

After 30 editions, add a cumulative section:
"Of N 🟡 alerts issued, X reverted to 🟢, Y became permanent frame shifts,
Z escalated to 🔴." This is the alert system's credibility engine.]

### Today's Framing-Shift Summary
[Moved here from end of section — the framing weather report is most useful
BEFORE the reader dives into per-story detail. Summarize today's alerts:
how many 🟢, how many 🟡, any 🔴. Highlight which 🟡 alerts are worth
tracking across editions.]

[Full output from media-analyst: Story-by-story positional readings
(with framing-shift alerts 🟢/🟡/🔴 per outlet per story, each outlet
getting a full analytical paragraph) → Framing alert summary table per story →
Apparent Contradictions → Overall Blind Spot.

Each outlet's positional reading must be a full analytical paragraph — not compressed
bullet-point treatment. The daily cadence justifies density in other sections, but
the media analysis is the product's moat: granularity here is non-negotiable.]

---

## SECTION 3 — From Framing to the FX Positions
[Mandatory when Sections 1 and 2 are both present. This section closes the loop
between media analysis and FX positioning by answering: "Who reads this frame,
and how does it move money?"

For each major framing divergence identified in Section 2, specify:
1. Which capital pool consumes this frame (Gulf SWFs, US institutional, macro HFs, etc.)
2. The positioning logic it encourages (de-dollarization, commodity-EM pairs, buy-the-dip)
3. The FX transmission mechanism (DXY pressure, VIX suppression, trade flow)
4. The net BRL direction

End with a convergence check: are all frames pointing the same direction? If yes,
flag the contrarian risk — uniform narrative support means the downside, when it comes,
hits all positioning simultaneously.

Include a summary table: Outlet | Frame | Actor | Mechanism | BRL Direction]

---

## SECTION 4 — 3-Point Stress-Test
[Full output from media-analyst's theory section:
Data Inputs (from Section 1) → What Each Theory Predicts →
Reality Check Table → Scorecard → Convergence → Divergence →
What This Tells the Reader]

---

## FORECAST
[Mandatory. Testable predictions synthesized from all preceding sections.
Each forecast must include: the prediction, confidence level, conditions that
would invalidate it, and a diagnostic test the reader can watch for.

Four forecasts are required every edition:

### BCB stance (next COPOM)
- Direction + magnitude (cut 25bps / hold / etc.)
- Confidence level
- Conditional logic: what changes the forecast?
- Surprise scenario with estimated probability

### Fed stance (next FOMC)
- Direction (hold / cut / hike)
- Tone variable: what language to watch for in the statement
- Next actual move estimate with conditions

### BRL/USD — 30-day range
- Central range with probability
- Scenarios table: base / pullback / bull / reversal — each with range,
  probability estimate, and trigger
- Asymmetry assessment: is the downside faster/steeper than the upside?

### Geopolitical wildcard
- The dominant binary risk right now (changes across editions)
- Probability assessment for each outcome
- Diagnostic test: what does the market's reaction to the resolution tell us?

Every forecast is a bet. State it clearly enough to be scored.]

---

## REVIEW — Where We Were Wrong
[Mandatory from edition 2 onward. Edition 1 establishes the baseline by
reviewing the pre-crisis consensus and where prevailing assumptions were wrong.

Structure:
1. **Scorecard**: Line-by-line assessment of prior edition's forecasts.
   For each: what we predicted → what happened → correct/wrong/partially right.
2. **Error analysis**: For each miss, explain *why* — was it a data gap,
   a model failure, or an unforeseeable shock?
3. **What the consensus got wrong**: Broader market consensus errors in the
   past 30 days, with analysis of whether our product caught or missed them.
4. **Cumulative accuracy**: After 10 editions, maintain a running accuracy
   table by category (BCB, Fed, BRL range, geopolitical). After 30 editions,
   publish the full track record.

The Review section is the product's credibility engine. Readers will forgive
wrong forecasts if the errors are analyzed honestly. They will not forgive
a product that never acknowledges its misses.]

---

## SOURCES
[Complete list of all sources consulted across all phases, with retrieval dates.
Organized by: Primary official sources → Market data → News outlets.
All primary source citations must include clickable hyperlinks where URL is known.
Flag any source that was unreachable or returned no relevant results.

**Live-blog link persistence**: For live blogs (CNN, Al Jazeera, Euronews),
add retrieval timestamp and note "(live blog; content rotates)". Suggest
Wayback Machine for archival verification.]

---

## FOR THE ECON STUDENT
[This section goes at the very end of the English version. Defines 5-8 key technical
terms in plain language (3-4 lines each), plus 3 reading suggestions with links.]

---

## EDIÇÃO EM PORTUGUÊS (Phase 5)
[Complete Portuguese translation of the entire report. Follow Phase 5 rules:
Brazilian number formatting, Valor Econômico register, proper noun preservation,
⟨derivado⟩ notation, identical hyperlinks. Structure mirrors the English version
exactly — same sections in the same order.]

---

## CROSS-REFERENCE CHECK (Phase 6)
[Table comparing every numerical data point across both versions:
Indicator | EN value | PT value | Match (✓/✗)
Must include: spot rate, SELIC, IPCA, DXY, Fed Funds, spread, CDS, IBOVESPA,
foreign inflows, fiscal deficit, VIX, oil, all SWF AUM estimates, all derived figures.
Report: "X/Y verified. Discrepancies: [list or none]."
Fix any discrepancy before publishing.]

---

## Inline Hyperlinks Rule (applies to entire report)

Every citation from a primary official source must include a clickable hyperlink.
This is non-negotiable for institutional credibility. Readers will click through.

## Source Tier System (applies to entire report)

Every citation carries a tier marker: T1 (primary issuer), T2 (licensed redistributor),
T3 (analytical estimate), T4 (secondary report). Format: `([Source](url) · TN, date)`.

Full definitions and source classifications are in `brl-usd-trader/references/
source-tier-classification.md`. The four assessment dimensions: granularity (how
frequently updated), source type (central bank vs. aggregator vs. journalism),
recency (how fresh the cited figure is), and aggregation transparency (how clearly
the source documents its methodology).

Rules:
- Prefer higher tiers. Cite T1 when available; use T2 when T1 is paywalled.
- Flag tier gaps: "free CDS sources lag ~2 weeks; real-time requires Bloomberg (T1)."
- Confirmation chains strengthen citations: `(Trading Economics · T2; confirmed
  Rio Times · T4 citing B3 · T1)` — the effective authority is T1.
- Derived figures inherit the tier of their weakest input.
- **Temporal freshness suffix (A–D)**: Every citation also carries a freshness marker.
  A = ≤7 days, B = 8–14 days, C = 15–21 days, D = 22+ days from report date.
  Full format: `([Source](url) · TN-X, date)`. Example: `([BCB](url) · T1-D, March 18)`.
  Derived figures inherit the freshness of their oldest input.
- The Source Tier Reference Table appears at the end of the Sources section,
  listing every source with its tier and dimensional breakdown.

## Figure Provenance Standard (applies to entire report)

Every number in the report is one of three types. The reader must be able to tell
which type *without clicking anything* — the provenance is inline, at point of use.

- **Sourced**: `value ([Source](url), date)` — the standard format.
- **Derived**: `value ⟨derived: formula using sourced inputs⟩` — formula visible inline.
- **Estimated**: `~value ([Estimator](url), date; entity does not disclose)` — estimator named, disclosure gap flagged.

This is a core product differentiator. Most research buries its assumptions in footnotes
or omits provenance entirely. Our standard: zero ambiguity about where every number
comes from, visible at the point the reader encounters it.

**Aggregation rule**: When a total is the sum of individually-sourced components
(e.g., "Gulf SWFs ~$2.4T combined" from ADIA + KIA + QIA), show the addition
inline: `~$2.4T ⟨derived: $1.1T + $750B + $510B⟩`. Never force the reader to
do arithmetic to verify consistency between a breakdown and its aggregate. If the
components appear in a different section, repeat them in the derivation tag.

Format: "[SELIC at 14.75%](https://www.bcb.gov.br/en/monetarypolicy/copomstatements)"
Not: "SELIC at 14.75% (BCB, March 18)"

Priority for hyperlinking:
1. COPOM statements/minutes (bcb.gov.br)
2. FOMC statements/dot plot (federalreserve.gov)
3. Focus survey (bcb.gov.br)
4. IPCA releases (ibge.gov.br)
5. IMF WEO/GFSR chapters (imf.org)
6. Specific news articles cited in media analysis

If a URL cannot be confirmed during the session, use parenthetical format. Never invent a link.

---

## Phase Execution Notes

### Phase 1: Data Collection (brl-usd-trader)
- Follow Step 1 exactly: US-side → Brazil-side → Global/Commodity
- Apply verification protocol: source hierarchy, timestamping, cross-verification
- Do NOT proceed to Phase 2 until all numbers are verified
- If a key data point cannot be verified from primary sources, flag it and note
  the best available source with date

### Phase 2: Media Analysis (media-analyst)
- Identify which stories are active today — typically 3-5 for a daily briefing
- For each story, search the relevant registry outlets
- Use Phase 1's verified data as ground truth — if a news outlet cites a different
  number, use Phase 1's figure and note the discrepancy
- Produce positional readings for each outlet on each story
- The "Apparent Contradictions" section is mandatory when data contradicts narrative

### Phase 3: Framing-to-FX Bridge
- Map each outlet's frame → capital pool → FX mechanism → BRL direction
- Build the Framing Risk Map table
- Check for convergence risk (all frames pointing same direction = contrarian risk)

### Phase 4: Assembly
- Apply 3-point stress-test using Phase 1 data + Phase 2 framing
- Build Forecast section (4 testable predictions with diagnostic tests)
- Build Review section (score prior forecasts or establish baseline)
- Write the Executive Brief LAST — it summarizes all sections
- First two sentences of the Executive Brief must work as a standalone social post
- Final verification pass: re-read the entire report and confirm every number
  carries source + date, every factual claim meets two-source minimum

### Phase 5: Portuguese Edition (Edição em Português)
- Translate the complete report into Brazilian Portuguese
- Apply Brazilian number formatting: decimal comma (4,997 not 4.997), period for
  thousands (1.112 bps not 1,112 bps), currency as R$ with comma decimal (R$59,8B)
- Translate the Figure Provenance notation: ⟨derived:⟩ → ⟨derivado:⟩,
  ⟨estimated:⟩ → ⟨estimativa:⟩
- Translate section headers but keep indicator names in their original form
  (SELIC, COPOM, FOMC, DXY, IBOVESPA, Focus stay untranslated — they are proper nouns)
- Translate analytical narrative with Brazilian financial register — not academic
  Portuguese, not colloquial. The target is Valor Econômico tone: precise, professional,
  assumes the reader knows macro but doesn't assume they read English comfortably
- Keep all hyperlinks identical to the English version — sources don't change
- The Econ Student section should reference Brazilian editions of readings where
  available (Furtado in the original Portuguese, Minsky in Portuguese translation if exists)
- Signal line translates: "BRL ALTISTA (médio) @ 4,997"
- "Bullish" → "Altista", "Bearish" → "Baixista", "Mixed" → "Misto"

### Phase 6: Cross-Reference Check
- After both versions are complete, extract every numerical data point from both
- Compare EN ↔ PT in a table: indicator, EN value, PT value, match (✓/✗)
- Formatting differences are expected and correct (4.997 vs 4,997) — flag only
  substantive discrepancies (different numbers, missing data, wrong direction)
- Verify that all ⟨derived⟩/⟨derivado⟩ formulas use identical arithmetic
- Verify that all source hyperlinks are identical
- Report result: "X/Y data points verified. Discrepancies: [list or none]"
- If any discrepancy is found, fix it in both versions before publishing

### Phase 7: Clean Machine Speak (post-generation, automated)
- Runs ~/bin/clean-report.py against the raw EN report
- Strips everything before the first "# " heading (preamble, process narration)
- Removes stray code fences, collapses excess whitespace
- Validates expected sections exist (EXECUTIVE BRIEF, SECTION 1, SECTION 2, FORECAST)
- Warns on placeholder text ([TBD], [INSERT], TODO:)
- If clean fails, falls back to raw report with a warning in the log

### Phase 8: Splice (post-clean, automated)
- Runs ~/bin/splice-report.sh against the cleaned EN report
- Extracts signal_line.txt (first # heading, stripped, ≤240 chars for BlueSky)
- Extracts exec_brief.md (full Executive Brief section for email)
- Builds exec_brief_social.txt (first 1-2 sentences, ≤400 chars for Mastodon)
- Copies full_report.md (complete cleaned report for website and Substack)
- All outputs go to a timestamped splice directory

### Phase 9: Distribute (post-splice, automated)
- 9a: Email — exec brief as HTML via Pandoc + msmtp through Proton Bridge
- 9b: GitHub Pages — git push full report to gh-pages branch (goes FIRST so links are live)
- 9c: Wait 45 seconds for GitHub Pages rebuild
- 9d: BlueSky — post signal_line.txt + edition URL via API (app password auth)
- 9e: Mastodon — post exec_brief_social.txt + edition URL via API (access token auth)
- 9f: Substack — manual for now (copy from archive, paste into editor)
- Each social channel fails independently and logs its own success/failure
- Distribution order matters: website must be live before social posts go out

---

## Scope Calibration

The full combined report is substantial. Calibrate depth to the user's request:

- **"Full report" / "in-depth" / "comprehensive"** → All nine phases, both languages
- **"Quick briefing" / "update"** → Executive Brief only + abbreviated Part 1 (EN only)
- **"What's happening today"** → Executive Brief + Part 2 headlines only (EN only)
- **"Should I buy dollars"** → Part 1 only (brl-usd-trader, EN only)
- **"How are outlets covering X"** → Part 2 only (media-analyst, EN only)
- **"Bilingual" / "Portuguese" / "both versions"** → Phases 5+6 mandatory

When in doubt, produce everything. The two-tier structure means the reader can
stop at the Executive Brief if they want the quick version.
