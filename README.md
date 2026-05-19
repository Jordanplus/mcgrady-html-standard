# mcgrady-html-standard

> A Claude Code skill that turns any reader-facing HTML — reports, dashboards, explainers, comparisons, briefings — into a single self-guiding HTML file — CSS-first, with inline SVG (d2 / mermaid) for complex diagrams — that a human can scan top-down without window-switching.

![version](https://img.shields.io/badge/version-0.2.0-blue)
![license](https://img.shields.io/badge/license-MIT-brightgreen)
![tier](https://img.shields.io/badge/tier-orchestration-yellow)
![trigger](https://img.shields.io/badge/trigger-%2Fmcgrady--html--standard-lightgrey)

---

## Contents

1. [What this is](#1-what-this-is)
2. [How to read this README](#2-how-to-read-this-readme)
3. [Install](#3-install)
4. [How to trigger it](#4-how-to-trigger-it)
5. [What the skill enforces](#5-what-the-skill-enforces)
6. [Quick before / after example](#6-quick-before--after-example)
7. [License & contributing](#7-license--contributing)

---

## §1 What this is

> **In one sentence** — Drop this skill into `~/.claude/skills/` and every HTML report you ask Claude Code to produce will have a TOC, a verdict banner, a "How to read this" routing table, per-chapter Reader's Guides, and a 4-layer expansion of every item you must act on — no CDN, no dispersed multi-file output, and JS is reserved for tier-2 interactive diagrams only.

This is a **Tier B (orchestration)** skill — a single `SKILL.md` file that Claude Code auto-loads and uses as a style guide whenever it is about to write or render an HTML page a human will read. There is no Python, no dependencies, no setup beyond placing the file. The skill applies across many reader-facing archetypes:

| Archetype | Examples |
|---|---|
| **Audit / judgment reports** | cross-vendor co-audit, code review summary, security review |
| **Progress snapshots** | OKR mid-quarter status, sprint review |
| **Benchmark results** | perf regression run, threshold-check report |
| **Retrospectives** | quarter-end retro, post-mortem |
| **Dashboards** | live ops / cost / SLA snapshots |
| **Explainers / how-tos** | onboarding docs, RFC walkthroughs |
| **Option-comparison briefs** | "which X should we pick" trade-off matrices |
| **Milestone briefings** | "what ships in M6" announcements |

Explicitly **not** for raw data dumps (raw logs, file diffs, JSON pretty-prints) — those are machine-readable and should stay raw.

---

## §2 How to read this README

| If you want to … | Jump to |
|---|---|
| Understand what this skill is in 30 seconds | [§1](#1-what-this-is) |
| Install it on your machine right now | [§3](#3-install) |
| See how to trigger it (auto vs manual) | [§4](#4-how-to-trigger-it) |
| Know what rules the skill will enforce on your HTML output | [§5](#5-what-the-skill-enforces) |
| See a before/after of raw output vs skill-shaped output | [§6](#6-quick-before--after-example) |
| Read the license / contribute | [§7](#7-license--contributing) |

---

## §3 Install

> **Why this matters** — No build, no dependency install. Just drop one Markdown file into your Claude Code skills directory. Total time < 1 minute.
>
> **What to focus on** — Four steps below. Step 3 is the only "magic" — Claude Code reads the YAML frontmatter and auto-loads the skill on its next session.
>
> **Bottom line** — Clone → copy → restart session → use.

**1. Clone this repo**

Anywhere on your machine — a scratch directory is fine.

```bash
git clone https://github.com/Jordanplus/mcgrady-html-standard.git
```

**2. Place the skill folder under your Claude Code skills directory**

User-scope skills live at `~/.claude/skills/`. Move the freshly-cloned folder into it.

```bash
mkdir -p ~/.claude/skills
mv mcgrady-html-standard ~/.claude/skills/
```

> **Project-scope alternative:** place it at `<your-project>/.claude/skills/mcgrady-html-standard/` if you only want it active inside one project.

**3. Claude Code auto-discovers the skill**

No config edit. On the next Claude Code session, the YAML frontmatter's `name` and `description` are indexed; the skill loads automatically whenever you ask Claude to produce a reader-facing HTML.

**4. Use it**

```bash
# auto: just ask for an HTML report — skill loads on description match
> produce an HTML review comparing baseline vs with-skill runs

# manual: explicit slash trigger
> /mcgrady-html-standard
```

<details>
<summary>Uninstall</summary>

```bash
rm -rf ~/.claude/skills/mcgrady-html-standard
```

Restart the Claude Code session. The skill is no longer indexed.

</details>

---

## §4 How to trigger it

| Trigger | When it fires | Example user prompt |
|---|---|---|
| ✅ auto | Claude is about to write or render an HTML file a human will read. | "produce an HTML report comparing these three runs" |
| ✅ auto | User asks for "dashboard / review / audit summary / explainer / comparison brief". | "draft a dashboard summarizing SLA over Q2" |
| ✅ auto | Claude is consolidating multiple JSON / log artifacts into one reader-facing surface. | "turn these 5 eval JSONs into a review" |
| ⬜ manual | You want to explicitly invoke the rules even if Claude did not auto-detect. | `/mcgrady-html-standard` |
| ❌ skip | Output is raw machine-readable data (logs, pretty JSON, file diffs). | "dump the raw test log to disk" |

---

## §5 What the skill enforces

> **Why this matters** — Before installing, know what the skill will change in Claude's HTML output. Nothing here is a surprise — the skill is opinionated on purpose.
>
> **What to focus on** — 7 rules below. The four-layer template (rule 4) is the most opinionated and only fires on items that need reader action.
>
> **Bottom line** — Single self-contained file, structured for top-down scanning; no CDN at any tier; JS only when an interactive diagram needs it.

1. **Single file, chapter-style.** No dispersed `baseline.html / withskill.html / diff.html`; one `review.html` with §1~§N chapters.

2. **Required 6-block structure.** Header → §1 Executive TL;DR (with verdict banner) → §2 How-to-read routing table → §3..§N body sections → Appendix (collapsed) → TOC with sticky back-to-top.

3. **Per-chapter Reader's Guide.** Every chapter opens with a 3-line guide: *Why this matters / What to focus on / Bottom line*. Tells the reader what to scan for before they dive in.

4. **Four-layer template for flagged items.** Any item where the reader is being asked to decide or act (a failure, a blocker, a threshold breach, a "stop" practice, an alert row) expands into *Context anchor / Trigger / Counterpoint / Action*. Routine rows (explainer steps, comparison options, normal-status rows) stay one-liners.

5. **Tiered visualization.**
   - Tier 0: ASCII / CSS for ≤3-node simple flows.
   - Tier 1: pre-rendered inline SVG (via Mermaid / d2 / graphviz) for ≥4 nodes or any branching — no runtime JS.
   - Tier 2: inline JS-lib bundle for zoom / pan / interactive — only when needed.
   - CDN forbidden at every tier (HTML must work offline, archived, emailed, printed).

6. **Caller-owned paths.** HTML lives inside the project the report is about (e.g. `your-project/reviews/review.html`) — not in `~/Desktop/` or any global stash.

7. **Output language follows the active `CLAUDE.md`.** The HTML's prose / labels / Reader's Guides are rendered in the language your `CLAUDE.md` specifies; technical terms, code, file names, and short universal status tokens (`PASS` / `FAIL` / `DONE`) stay in their original form.

For the full ruleset (taxonomy table, four-layer template details, ASCII flow diagrams, anti-patterns), see [`SKILL.md`](./SKILL.md) in this repo.

---

## §6 Quick before / after example

> **Why this matters** — Concrete shows the shape difference better than abstract rules. Same input data, two outputs.
>
> **What to focus on** — The "after" version answers *what / so-what / now-what* in the first 2 seconds. The "before" version forces the reader to assemble that themselves.
>
> **Bottom line** — The skill turns a data dump into a scannable verdict.

<details open>
<summary>Before: raw output, no skill</summary>

```html
<h1>Eval results</h1>
<p>ran 3 vendors x 14 assertions = 42 judgments</p>
<table>
  <tr><th>ID</th><th>claude</th><th>openai</th><th>gemini</th></tr>
  <tr><td>1/[1]</td><td>PASS</td><td>PASS</td><td>PASS</td></tr>
  <tr><td>3/[2]</td><td>FAIL</td><td>FAIL</td><td>FAIL</td></tr>
  ...
</table>
<p>3/[2] needs review</p>
```

Reader has to scan the whole table to find what's wrong, has no idea why `3/[2]` needs review, and has no path to action.

</details>

<details open>
<summary>After: skill-shaped</summary>

**§1 Verdict** — 1 regression introduced (`3/[2]`: Posted Header Credits coverage missing). 41 of 42 judgments PASS. Patch recommended — all 3 judges agree.

…followed by §2 routing table, §3..§N chapter bodies, and for entry `3/[2]`:

```
3/[2] Report mentions Posted Header Credits=4 ...  [INTRODUCED]
  🧭 Context anchor:
    · Original input: "Analyze PCIe credit-based flow control timeout risks"
    · Issue: audit on PCIe section of the report skill
    · Current assertion: must mention Posted Header Credits=4 as bottleneck
  🔍 Trigger: claude's response (FAIL)
    judge[claude]:  FAIL — "...never mentions Posted Header Credits=4"
    judge[openai]:  FAIL — "...discusses NPD=0 but not PH Credits=4"
    judge[gemini]:  FAIL — "...does not analyze 'Posted Header Credits=4'"
  📌 Counterpoint: openai's earlier run (PASS) covered:
    > Posted Header credits = 4 ... can cause posted-write backpressure
  🛠 Action:
    (a) Patch the PCIe section to add the keyword "Posted Header Credits"
    (b) Accept if only one judge dissents — here all 3 FAIL → strong signal, patch
    (c) Concrete: add a risk entry to docs/pcie_timeout_risks.md; re-run audit
```

Reader sees the verdict in 2 seconds; knows what flagged it, what "good" looks like, and what to do.

</details>

---

## §7 License & contributing

**License:** MIT. See [`LICENSE`](./LICENSE) for the full text. You are free to use, modify, redistribute, and embed this skill in your own pipeline.

**Contributing:** issues and pull requests welcome on GitHub. Suggested change areas:

- Additional report archetypes (anything you find yourself producing that does not fit the existing taxonomy)
- More worked examples (the canonical examples cover audit, progress, benchmark — retrospective and dashboard examples are missing and very welcome)
- Translations of structural labels (the skill says "follow the active CLAUDE.md", but having canonical translations for common languages would shorten the path)
- Better visual aids inside `SKILL.md` (the existing ASCII diagrams work but improvements welcome)

**Version history:**

| Version | Changes |
|---|---|
| `0.2.0` | Added tiered diagram model (Tier 0 ASCII / CSS, Tier 1 pre-rendered inline SVG via Mermaid / d2 / graphviz, Tier 2 inline JS-lib bundle for interactive) |
| `0.1.0` | Initial release |

Breaking changes to the four-layer template names will bump the major version; additive examples / archetypes bump minor.
