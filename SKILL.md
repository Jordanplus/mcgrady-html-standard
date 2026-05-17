---
name: mcgrady-html-standard
description: Conventions for producing any reader-facing HTML a human is expected to scan top-down — reports, dashboards, explainers / how-to pages, option-comparison briefs, milestone briefings, retrospectives, benchmarks, audits. NOT for raw data dumps (raw logs / file diffs / JSON pretty-prints), which are machine-readable. Loads automatically when the assistant is about to write or render an HTML file the user will read, when the user asks to "produce an HTML report / review / dashboard / explainer / comparison brief", or whenever consolidating multiple JSON / log artifacts into a reader-facing surface. Enforces: single-file chapter layout (not dispersed multi-file), required structure (Header / Exec TL;DR / How-to-read / sections / Appendix / TOC), per-chapter Reader's Guide (3-line "why this matters / what to focus on / bottom line"), the four-layer template (context anchor / trigger / counterpoint / action) for items the reader is asked to act on — flagged failures, blockers, threshold breaches, "stop" practices, alert rows — but NOT for routine entries (explainer steps, comparison options, normal-status rows), tiered visualization (ASCII / CSS for ≤3-node simple; pre-rendered inline SVG via Mermaid / d2 / graphviz for ≥4 nodes or branching; inline JS-lib bundle for interactive — CDN forbidden at every tier), caller-owned file paths, and explicit anti-patterns.
version: 0.2.0
tier: orchestration
trigger: /mcgrady-html-standard
---

# mcgrady-html-standard — HTML Report Output Standards

**When this applies**: any HTML you produce for a human to read top-down — reports, dashboards, explainers / how-to pages, option-comparison briefs, milestone briefings, retrospectives, benchmarks, audits. **NOT** for raw data dumps (raw log files, file diffs, JSON pretty-prints), which are machine-readable.

Quick decision before reading further:

```
    Am I writing HTML?
    │
    ├─ Is the reader a human scanning top-down?
    │   │
    │   ├─ NO (logs, JSON pretty-print, file diffs for tooling)
    │   │      → this skill does NOT apply. Skip.
    │   │
    │   └─ YES
    │       │
    │       └─ Does the output consolidate multiple sources / make
    │          a judgment / compare runs / explain something / brief
    │          a milestone / show a dashboard?
    │           │
    │           ├─ NO (single trivial table, one-shot status string)
    │           │      → overkill. Skip; render plain HTML.
    │           │
    │           └─ YES  → APPLY this skill
    │                  · single file, chapter-style
    │                  · required 6 blocks (Header / §1 / §2 / §3..§N / Appendix / TOC)
    │                  · per-chapter Reader's Guide
    │                  · flagged item (needs reader action) → 4-layer template
    │                  · CSS-only, caller-owned path
```

## Reader-facing archetypes — pick the one that matches before going further

The structural backbone (single-file, 6-block layout, Reader's Guide, anti-patterns, CSS-only skeleton) is universal across all reader-facing HTML — keep it as the default. The four-layer template (defined later in this document) is a **specialization** that only kicks in when the reader is being asked to act on an item.

| Archetype | Typical example | Items needing reader action? | Apply four-layer template? | What you keep from this skill |
|---|---|---|---|---|
| Audit / judgment | cross-vendor co-audit | yes (failures / dissents) | yes — every flagged row | backbone + audit chips + template |
| Progress snapshot | OKR mid-quarter status | yes (blockers / at-risk) | yes — for blocked / at-risk rows | backbone + progress chips + template |
| Benchmark | perf regression run | yes (threshold breaches) | yes — for over-limit metrics | backbone + benchmark chips + template |
| Retrospective | quarter-end retro | yes (failed practices) | yes — for "stop" practices | backbone + retro chips + template |
| Dashboard | live ops / cost / SLA | sometimes (alert rows only) | only for alert rows | backbone; template only when alerted |
| Explainer / how-to | onboarding doc, RFC walkthrough | no | no | backbone + Reader's Guide only |
| Option comparison | "which DB" brief | no (just trade-offs) | no | backbone + per-option trade-off table |
| Milestone briefing | "what ships in M6" | no | no | backbone + Reader's Guide |
| Raw data dump | full log / pretty JSON | n/a — machine-readable | n/a | **out of scope** for this skill |

If your output is not on this table, default to "backbone + Reader's Guide" and apply the four-layer template wherever you flag an item the reader must act on.

## Output language follows the active CLAUDE.md

The HTML's user-visible text — title, headers, Reader's Guide, prose, action options, anti-pattern fixes, explanatory paragraphs — is written in the language specified by the **most specific active CLAUDE.md** (subdir CLAUDE.md > project-root CLAUDE.md > user-global CLAUDE.md). Technical terms, code identifiers, filenames, command names, API / library references, and short universal status tokens (`PASS` / `FAIL` / `DONE` / `WIP` / `OVER-LIMIT` / etc.) stay in their original form.

Rules:

- **This SKILL.md is in English regardless of project language.** It is rule-documentation Claude reads, not output a human reads. Do not translate the canonical templates / examples here; translate them only when rendering the final HTML.
- **If no CLAUDE.md language directive is present**, match the language the user used in the current task. Default to English if unclear.
- **Mixed-language directives** (e.g., "繁體中文, keep technical terms in English") are honored as stated: prose translates, code / API / file names stay English.

Example: under a `繁體中文 (Taiwan)` directive, the canonical English Reader's Guide

```
Why this matters: one sentence
What to focus on: 2~4 bullets
Bottom line: one sentence judgment
```

is rendered in the HTML as

```
為什麼看這段: 一句說明
關注什麼: 2~4 重點
結論: 一句判斷
```

The structural rule (three lines, fixed order: why / focus / bottom) is preserved; only the wording is translated.

## Core principle: one file, chapter-style, self-guiding

Do not split your output across multiple HTML files just because the material is dense — that forces readers to switch windows and rebuild context. **Default to a single integrated HTML report**, sectioned internally. Multi-file output is the exception (e.g. a dataset so large that single-file load is genuinely a problem).

```
❌ baseline_review.html + withskill_review.html + diff_review.html  (3 dispersed files)
✅ review.html  (with §1~§N chapters, TOC + sticky back-to-top)
```

## Required structure

Every review HTML must contain:

1. **Header**: title + generation timestamp + source-file paths (so the reader knows which run this is)
2. **§1 Executive TL;DR**: 1~3 sentences + large verdict banner — the final conclusion
3. **§2 How to read this report**: table mapping "questions the reader will have → which chapter to read" (reader navigation)
4. **Body sections**: multiple `<section>` blocks, each with an `<h2>` — the core data and evidence
5. **Appendix**: raw data / full detail (default collapsed)
6. **TOC**: top-of-page table of contents with anchor links; for long reports add a `position: fixed` "↑ back to TOC" button

The reading flow this structure produces:

```
            ┌──────────────────────────────────────────┐
            │  Header   title · timestamp · sources    │  ← orient: which run is this
            ├──────────────────────────────────────────┤
            │  §1  Executive TL;DR + verdict banner    │  ← bottom-line first
            ├──────────────────────────────────────────┤
            │  §2  How to read this report             │  ← reader self-routes
            │      [question]  →  [chapter]            │
            ├──────────────────────────────────────────┤   ←──┐
            │  §3  Chapter — Reader's Guide            │      │
            │      Why · Focus · Bottom line           │      │
            │      [ body: tables / charts / cards ]   │      │ reader scans
            ├──────────────────────────────────────────┤      │ top-down,
            │  §4  Chapter ...                         │      │ but can also
            │      Reader's Guide + body               │      │ jump from §2
            ├──────────────────────────────────────────┤      │ routing
            │  ...                                     │      │
            ├──────────────────────────────────────────┤      │
            │  §N  Appendix (collapsed by default)     │   ←──┘
            │      raw / per-item / oversized detail   │
            └──────────────────────────────────────────┘
                       ▲
                       │
              TOC nav (sticky) + ↑-to-top button
```

## Every chapter MUST carry a Reader's Guide

Every main chapter (from §3 onward) opens with this three-line guide box:

```
┌─────────────────────────────────────────────┐
│ Why this matters: one sentence              │
│ What to focus on: 2~4 bullets               │
│ Bottom line: one sentence judgment          │
└─────────────────────────────────────────────┘
```

Do not assume the reader knows what each table / metric means. **"Why this matters" anchors them; "What to focus on" steers attention; "Bottom line" relieves the burden of re-reading the whole chapter.**

## Items the reader must act on must expand into the four-layer template

**Any item where the reader is being asked to decide or act — a flagged failure, an introduced regression, a blocker, a threshold breach, a "stop" practice in a retro, an alert row in a dashboard — must carry the following four layers. Routine rows (an explainer step, a comparison option, a normal-status dashboard row, an informational briefing item) do not need this expansion; just put them in the body of the relevant section.**

The four layers nest like this — layer 0 wraps the rest, and a reader opening the card reads them in order:

```
    ┌──────────────────────────────────────────────────────────┐
    │ 0 · CONTEXT ANCHOR                                       │
    │     · original input / prompt                            │
    │     · issue / expected output                            │
    │     · the current assertion / fact                       │
    │   → reader knows where they are before anything else     │
    │                                                          │
    │   ┌────────────────────────────────────────────────────┐ │
    │   │ 1 · TRIGGER   [default open]                       │ │
    │   │     who/what flagged it · why · raw explanation    │ │
    │   └────────────────────────────────────────────────────┘ │
    │                                                          │
    │   ┌────────────────────────────────────────────────────┐ │
    │   │ 2 · COUNTERPOINT                                   │ │
    │   │     a passing / reference / on-track sample        │ │
    │   │     → shows what the trigger missed                │ │
    │   └────────────────────────────────────────────────────┘ │
    │                                                          │
    │   ┌────────────────────────────────────────────────────┐ │
    │   │ 3 · ACTION                                         │ │
    │   │     (a) fix option   (b) accept option             │ │
    │   │     (c) escalate rule + concrete next step         │ │
    │   └────────────────────────────────────────────────────┘ │
    └──────────────────────────────────────────────────────────┘
```

### Pick the column that matches your report

The 4 layers stay the same. Only the vocabulary inside them changes per report type:

| Layer | Audit / judgment | Progress snapshot | Benchmark | Retrospective |
|---|---|---|---|---|
| 0 Context anchor | spec + assertion + issue | workstream + milestone + dependency | workload + config + threshold | period + practice + outcome |
| 1 Trigger | judge voted FAIL + reasoning | task slipped + symptom | metric breached + delta | practice failed + consequence |
| 2 Counterpoint | another vendor PASSed + evidence | sibling workstream on track | baseline run within budget | a different practice that held |
| 3 Action | patch / accept / escalate | re-plan / re-staff / de-scope | tune / re-bench / accept regression | keep / start / stop |

The canonical worked example below uses the **Audit** column (preserving the PCIe Posted Header Credits walkthrough). Two shorter examples after it illustrate the Progress and Benchmark columns.

0. **Context anchor (issue / spec / scenario the entry refers to)**: readers will not automatically remember which spec / scenario this case is about from the title alone. Must include:
   - **Original input / prompt**: what the user originally asked, what spec snippet was fed in, what the scenario was
   - **Issue / expected output**: what this case was verifying overall (not just the current single assertion)
   - **The current assertion / fact**: the single requirement being checked in this entry (which is one facet of the larger issue)
   - All three layers stacked let the reader fully locate: who they are looking at, what is being discussed, and where the current assertion sits within the bigger picture.
1. **Trigger / signal source**: what caused this entry to be flagged. For an audit: which vendor / judge / module voted FAIL and why. For a progress snapshot: which task slipped and the symptom. For a benchmark: which metric breached and by how much. For a retrospective: which practice failed and the consequence.
   - Show the trigger's raw explanation (multiple judges → list each one, each with its verdict chip; multiple symptoms → list each)
   - **Default expanded** (`<details open>`), do not collapse — this is what the reader needs to see immediately
2. **Counterpoint / reference sample**: find a "passing / on-track / within-budget / kept-practice" counter-example
   - e.g. another vendor that gave PASS on the same assertion; a sibling workstream on track with the same milestone; a baseline run within budget; a different practice that held
   - Lets the reader see directly "what the trigger missed" or "what good looks like" in the same conditions
3. **Action options** (explicit, pickable):
   - (a) Fix-it option: e.g. "patch SKILL.md / re-plan workstream / tune config / keep this practice"
   - (b) Accept option: e.g. "treat as noise, document and ignore" / "accept the regression and re-baseline"
   - (c) Escalation rule: e.g. "if N items all fail similarly → strong signal, must patch / de-scope / re-staff"
   - End with a "concrete next step" one-liner: e.g. "add X under Y, re-run to verify" / "post diff to channel Z by EOW"

### Why the context anchor (layer 0) is mandatory

An ID alone (e.g. `3/[2]`) has no mnemonic value for humans; the assertion text (e.g. "report mentions Posted Header Credits=4 could become a bottleneck") is only "one facet of the issue", and without spec context the reader cannot tell if it is about PCIe, USB, or DDR. After reading the trigger + action segments, readers often ask "wait, which spec / scenario / context is this again?" — by then they have to flip back to the source artifacts (eval JSON, project tracker, raw metric log, retro notes — whatever fed this report) and break the train of thought.

**So the context anchor must be written before the trigger block**, letting the reader build full context the moment they open the card, then read the rest in order.

**Anti-example**: `"3/[2] introduced — needs review"` — reader cannot see who failed, why, or what to do.

**Correct example**:
```
3/[2] Report mentions Posted Header Credits=4 ...  [INTRODUCED]
  🧭 Context anchor:
    · Original input: "Analyze PCIe credit-based flow control timeout risks in our IP"
    · Issue: cross-vendor co-audit on PCIe section of SKILL.md
    · Current assertion: report must mention Posted Header Credits=4 as a bottleneck candidate
  🔍 Trigger: claude's response (Level-1: FAIL)
    judge[claude]: FAIL — "The response focuses on Non-Posted credits, never mentions Posted Header..."
    judge[openai]: FAIL — "The response discusses NPD=0 but does not mention Posted Header Credits=4..."
    judge[gemini]: FAIL — "The assistant's response does not analyze the 'Posted Header Credits=4' parameter..."
  📌 Counterpoint: openai's response (PASS) covered the concept here:
    > Posted Header credits = 4, Posted Data credits = 16 are small; can cause posted-write backpressure
  🛠 Action:
    (a) Patch the SKILL.md PCIe section to include the keyword "Posted Header Credits"
    (b) Accept: treat as one-off if only one judge dissents — but here all three FAIL → strong signal, patch
    (c) Concrete: add a risk entry to references/ip-database/pcie_timeout_risks.md explicitly requiring PH-credits coverage; re-run audit to verify
```

**Progress snapshot example** (a slipped workstream):
```
WS-7 / Search ranking refresh                          [BLOCKED]
  🧭 Context anchor:
    · Original input: 2026-Q2 OKR — refresh ranking model with new behavior signals
    · Issue: ship v2 ranker behind A/B by end of M5
    · Current row: M4 checkpoint says feature pipeline is gated on schema review
  🔍 Trigger: data-platform team has not closed schema review (3 weeks open)
    Eng-Mgr Ivy flagged in standup; M4 checkpoint missed by 8 days
  📌 Counterpoint: WS-3 (CTR re-baseline) hit its M4 checkpoint
    Used the same schema process, closed in 5 days because they pre-circulated the diff
  🛠 Action:
    (a) Pre-circulate the schema diff to data-platform this week (proven path from WS-3)
    (b) Accept: re-plan M5 to M5.5 and de-scope (only if pre-circulation is refused)
    (c) Concrete: post the diff in #data-platform tomorrow, ask for review by EOW
```

**Benchmark example** (a metric breach):
```
p99 query latency / payments-api                       [OVER-LIMIT]
  🧭 Context anchor:
    · Workload: replay of 2026-W18 prod traffic
    · Config: build a1b2c3d, region us-east-1, 8 vCPU
    · Threshold: p99 ≤ 180 ms (set in OKR doc)
  🔍 Trigger: measured p99 = 247 ms (+37% vs threshold, +52% vs last week's baseline 162 ms)
  📌 Counterpoint: p50 = 38 ms (unchanged), p95 = 91 ms (+4%)
    Only the tail is regressed — not a uniform slowdown, so suspect a long-tail-only path
  🛠 Action:
    (a) Profile the long-tail requests; suspect a recent retry-policy change (PR #4421)
    (b) Accept: declare a new threshold of 220 ms (only if profiling shows fundamental cause)
    (c) Concrete: bisect between baseline build and a1b2c3d on the slowest 1% of queries
```

## Visualization preferences

### Diagrams — three tiers, pick by complexity

The skill previously mandated CSS-only / no-JS for everything, which forced complex flows into illegible ASCII art. Replaced with a tiered model:

| Tier | When to use | Tool | How it enters HTML | Runtime JS? |
|---|---|---|---|---|
| 0 — ASCII / CSS | ≤3 nodes, linear flow, simple list-style | hand-written `<pre>` ASCII or CSS box | inline text | no |
| 1 — Static inline SVG | ≥4 nodes, OR any branching / merging / cycle, OR ≥2 levels of nesting, OR state machine / DAG meant to be read top-down once | Mermaid CLI (`mmdc`), d2, or graphviz → SVG | paste full `<svg>...</svg>` into HTML body | no |
| 2 — Interactive | needs zoom / pan / collapse-on-click / hover-expand / live data swap | Mermaid client-side, ECharts, D3 | `<script>` block with the **entire lib bundle inlined** | yes |

**Upgrade rule**: a diagram with **≥4 nodes OR any branch / merge / cycle** MUST be at least tier 1 — do not leave it as ASCII. Linear 3-step "A → B → C" stays at tier 0; that level of complexity does not justify a build step.

**CDN is forbidden at every tier.** Reports get archived, emailed, opened offline, exported to PDF; CDN dependencies break in all of those flows. For tier 2, inline the entire library bundle into the HTML:

```html
<script>/* full content of mermaid.min.js pasted here */</script>
<div class="mermaid">flowchart LR ...</div>
<script>mermaid.initialize({startOnLoad: true});</script>
```

This makes the HTML larger (Mermaid minified ≈ 2.8 MB), so **do not promote a tier-0 / tier-1 diagram to tier 2 unless the interactivity is actually needed**.

**Local install** (one-time, macOS):

```bash
brew install d2
npm install -g @mermaid-js/mermaid-cli
npx puppeteer browsers install chrome-headless-shell  # mmdc needs headless chromium; postinstall does not pull it automatically
```

**How to generate tier-1 SVG** (do this before writing the HTML):

```bash
# Mermaid — default for flowcharts / sequence / state / class diagrams
mmdc -i diagram.mmd -o diagram.svg -b transparent

# d2 — architecture / layered system diagrams; rounded corners + curved edges by default
d2 diagram.d2 diagram.svg

# Graphviz — large DAGs; rounded boxes via style=rounded, curved edges via splines=curved
dot -Tsvg diagram.dot -o diagram.svg
```

**Recommended repo layout** when a report owns more than ~3 diagrams: put sources next to the HTML in a `diagrams/` sibling directory, with a `render.sh` that loops over `*.d2` / `*.mmd` and re-inlines into the HTML via `data-src="<basename>"` placeholders. This makes diagram edits a one-line re-run instead of manual SVG copy-paste.

Then paste the resulting SVG (including the `<svg>` wrapper) directly into the HTML body. Do NOT use `<img src="diagram.svg">` — that creates an external file dependency and breaks single-file portability.

### Other visual elements

- **Bar charts in pure CSS**: simple table + a `width: X%` div fill; promote to tier 2 (ECharts / Plotly inline) only if you need hover tooltips or live data swap
- **Verdicts / status as chip pills**: color semantics stay constant across report types (green = good / safe, amber = attention, red = problem, neutral / gray = inconclusive); the labels change per report type:

  | Report type | Default chips |
  |---|---|
  | Audit / judgment | green=PASS, amber=PARTIAL, red=FAIL, neutral=DISAGREE |
  | Progress snapshot | green=DONE, amber=WIP, red=BLOCKED, neutral=AT-RISK |
  | Benchmark | green=WITHIN-BUDGET, amber=NEAR-LIMIT, red=OVER-LIMIT |
  | Retrospective | green=KEEP, amber=START, red=STOP |
- **Shift / delta with arrows**: `baseline → with-skill` is intuitive
- **Long text / detail in `<details>` collapsed**: default closed, with a summary worth clicking

## File characteristics

- **Fully self-contained**: CSS inline in `<head><style>`; any tier-2 JS library bundled inside `<script>`; no external CDN, no remote fonts, no `<img src="…svg">` pulling separate diagram files
- **Core navigation works without JS**: TOC, anchors, `<details>` collapse, sticky back-to-top — all HTML/CSS native. JS is reserved for **tier-2 interactive diagrams / charts** and must be inlined (never CDN-loaded)
- **Semantic markup**: use `<section>` / `<article>` / `<header>` / `<nav>` — friendlier to print / screen readers
- **Print-friendly**: avoid fixed-position elements that block content when printed

## Paths and naming

- General convention: place inside the **caller's** own repo, **not in a global location**
  - ✅ `IP_review_skill/coeval/review.html`  (caller-owned)
  - ❌ `~/claude/coeval-reports/X_review.html`  (global location, violates caller-owned)
- Filenames must be semantically meaningful: `review.html` / `M6_baseline_review.html` / `2026-05-17_release_audit.html`
- For multiple versions, **put the date or milestone in the filename**, do not rely on modify time

## After writing

1. **Open in browser to verify**: use `open <path>` and read through it yourself to confirm chapter order makes sense and the reader's guides actually guide
2. **Tell the user the path + open command explicitly**: write out `open <path>`, do not just say "HTML produced"
3. **If replacing multiple older HTMLs**: explicitly say in the message "the following N older HTMLs have been superseded by review.html; you can keep them as raw or delete"

## Anti-patterns (do not do these)

Each row pairs the anti-pattern with the fix:

| ❌ Anti-pattern | ✅ Fix |
|---|---|
| Multiple independent HTMLs (one per aspect) → reader has to window-switch and rebuild context | One `review.html` with §1~§N chapters + TOC; reader scans top-down, no window switching |
| Data tables without a Reader's Guide → reader cannot tell which numbers matter | Each chapter opens with Why / Focus / Bottom line — reader knows which numbers matter |
| External CDN dependency (CSS framework, web font, JS lib loaded from unpkg / cdnjs / jsdelivr) → breaks when offline / archived / printed, leaks access patterns | Inline `<style>` in `<head>`, fonts from system stack; for tier-2 diagrams paste the full minified lib bundle into an inline `<script>` block — never reference a remote URL |
| Forcing a ≥4-node or branching diagram into ASCII art because "no JS allowed" → diagram becomes illegible spaghetti | Promote to tier 1: pre-render via Mermaid / d2 / graphviz and inline the resulting `<svg>` into the body; tier-0 ASCII is reserved for ≤3-node linear flows |
| Promoting every diagram to tier 2 (inline Mermaid + JS) for visual appeal → HTML balloons to 3+ MB, slow to print / email | Use tier 0 (ASCII) or tier 1 (static SVG) by default; only go tier 2 when zoom / pan / collapse / live-data is actually required |
| Flattening all detail at once → reader is drowned in information on the first paint | Headline / chip / one sentence on first paint; full detail behind `<details>` in Appendix |
| Placing HTML in `~/Desktop/` or other global location → no version control, cannot find prior runs | Caller-owned path inside caller's repo; filename carries date / milestone |

---

## Quick reference: a minimal review.html skeleton

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>X Review — Y</title>
<style>
  /* inline CSS only, no CDN */
  body { font-family: -apple-system, sans-serif; max-width: 1080px; margin: 0 auto; padding: 0 24px; }
  .reader-guide { background: #eef4fb; padding: 12px 16px; border-radius: 8px; margin: 12px 0; }
  .chip-pass { background: #e7f5ec; color: #1f7a3a; padding: 2px 9px; border-radius: 11px; }
  /* ... chip-fail / chip-partial / etc. */
  .toc { position: sticky; top: 0; }
  .back-to-top { position: fixed; bottom: 20px; right: 20px; }
</style>
</head>
<body>
<header><h1>...</h1><div class="meta">generated ... | source: ...</div></header>

<nav class="toc" id="toc"><h3>Contents</h3><ol>
  <li><a href="#exec">§1. Executive TL;DR</a></li>
  <li><a href="#howto">§2. How to read this report</a></li>
  <!-- ... -->
</ol></nav>

<section id="exec"><h2>§1. Executive TL;DR</h2>
  <!-- verdict banner + 1~3 sentences -->
</section>

<section id="howto"><h2>§2. How to read this report</h2>
  <!-- question → chapter routing table -->
</section>

<section id="..."><h2>§3. ...</h2>
  <div class="reader-guide">
    <div>Why this matters: ...</div>
    <div>What to focus on: ...</div>
    <div>Bottom line: ...</div>
  </div>
  <!-- body -->
</section>

<!-- ... more sections ... -->

<section id="appendix"><h2>§N. Appendix</h2>
  <details><summary>raw per-item details</summary>...</details>
</section>

<a href="#toc" class="back-to-top">↑ TOC</a>
</body>
</html>
```
