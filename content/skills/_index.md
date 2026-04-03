---
title: "Claude Skills"
description: "Reusable Claude Code skills for applied research — free to copy and use."
---

[Claude Code](https://docs.anthropic.com/en/docs/claude-code) is Anthropic's CLI tool for working with Claude on coding and research tasks. **Skills** are reusable instruction sets that teach Claude how to handle specific workflows — think of them as recipes that trigger automatically when you describe a task.

The skills below were developed through real research projects. They encode hard-won lessons about what works (and what doesn't) when using AI for academic research. Copy the skill file into your `~/.claude/skills/` directory and Claude Code will use it automatically.

---

<details>
<summary><strong>Conference Paper Roundup</strong> — Scrape, summarize, and blog about conference papers</summary>

**What it does:** Takes a conference programme URL and produces structured one-sentence summaries of every paper, grouped by topic, ready for a blog post. Enforces a strict 4-phase pipeline (Inventory → Download → Summarize → Verify) that prevents the most common failure modes.

**Born from:** The [CSAE 2026 roundup](/posts/csae-2026-roundup/) on this site, which summarized 206 papers. The first attempt took dozens of prompts because of avoidable errors (fabricated URLs, positional matching failures, silent agent timeouts). This skill encodes the fixes.

**Key lessons baked in:**
- Extract ALL metadata before downloading anything
- Match papers to links by title, not by position on the page
- Use Python `pdfplumber` for PDF extraction, not web fetchers
- Speaker ≠ first author — always get speaker names from the programme
- Store abstract + detailed summary + one-liner for each paper (allows iteration without re-reading)

**Install:**
```bash
mkdir -p ~/.claude/skills/conference-paper-roundup
# Copy the skill file below into:
# ~/.claude/skills/conference-paper-roundup/skill.txt
```

<details>
<summary>View full skill file</summary>

```
# Conference Paper Roundup Skill

## When to trigger
Use this skill when the user asks to summarize papers from a conference,
workshop, or similar event — especially when papers are available as PDFs
on a programme page.

## Critical Pipeline: 4 Phases in Strict Order

NEVER start a later phase until the previous phase is 100% complete.

### PHASE 1: INVENTORY
1. Navigate to the programme page
2. Extract ALL paper titles, speakers, session names, and PDF URLs in one pass
3. Save to all_paper_links.json
4. Confirm count with user before proceeding

Rules:
- Scrape ALL days/sessions in one go
- NEVER fabricate URLs (conference sites use random hashes)
- Match papers to links by TITLE, not by position

### PHASE 2: DOWNLOAD
1. Use Python requests + pdfplumber to download and extract text
2. Save extracted text to extracted_texts/paper_NNN.txt
3. Report success/failure count

Rules:
- Use a single Python script, not multiple agents
- Extract first 5 pages only
- Handle .docx files with python-docx

### PHASE 3: SUMMARIZE
For each paper, create a JSON with:
- title, authors, authors_cite, authors_detail (name, institution, country)
- speaker (from PHASE 1 programme, NOT from the paper)
- abstract (verbatim), detailed_summary (2-3 paragraphs), one_liner
- topic, countries_of_study, is_rct, pdf_url

Rules:
- Speaker comes from the programme, not the paper
- Store 3 levels of detail to allow iteration
- Process in batches via Python script, not agents

### PHASE 4: VERIFY
1. Check for missing fields, wrong speakers, duplicates
2. Cross-check against programme ground truth
3. Produce verification report

## One-Liner Style
- Lead with the finding, not the method
- Include country/context
- Have personality
- End with (Author et al.) and #RCT if applicable

Good: "Phone surveys systematically distort responses by up to 63%
compared to in-person interviews in rural Nigeria (Castaing et al.). #RCT"

Bad: "This paper uses an RCT to study survey mode effects in Nigeria."
```

</details>

</details>

---

<details>
<summary><strong>Data Wrangling</strong> — Systematic data cleaning and validation for research datasets</summary>

**What it does:** Runs a structured diagnostic checklist on any dataset (CSV, Excel, Stata .dta, R .rds) — checking for missing values, outliers, duplicates, skip pattern violations, merge quality, and panel consistency. Flags problems before fixing them and produces a numbered diagnostic report.

**Key principles:**
- **Print everything.** Every check produces visible output — the user never has to trust "it looks fine"
- **Non-destructive first pass.** Flag problems before fixing them
- **Preserve provenance.** Log every change for replication

**Diagnostic checklist:**
1. Structure and metadata (dimensions, types, ID uniqueness)
2. Missing values (patterns, coded-missing conversion, differential by treatment)
3. Outliers (statistical flags + substantive flags in context)
4. Skip patterns and logical consistency
5. Merge diagnostics (m:1 vs 1:1, unmatched observations)
6. Panel consistency (balanced panel, attrition, time-varying invariants)
7. Treatment variable integrity (constant within unit, mutually exclusive)

**Install:**
```bash
mkdir -p ~/.claude/skills/data-wrangling
# Copy the skill file below into:
# ~/.claude/skills/data-wrangling/SKILL.md
```

<details>
<summary>View full skill file</summary>

```
# Data Wrangling Skill

Default to R unless the user specifies otherwise.

## Core Principles
1. Print everything — every check produces visible output
2. Non-destructive first pass — flag before fixing
3. Report structure — numbered diagnostic report, pass/fail summary at top
4. Preserve provenance — log what changed, where, and why

## Diagnostic Checklist (in order)

### 1. Structure and Metadata
- Dimensions, variable names, types
- ID uniqueness (test explicitly)
- Duplicate rows and near-duplicates

### 2. Missing Values
- Count and percentage per variable
- Patterns: concentrated in certain groups/periods?
- Coded missing (-99, 999, "N/A") → convert and report
- Panel: differential missingness by treatment/round?

### 3. Outliers
- Summary stats with percentiles (1st, 5th, 95th, 99th)
- Statistical flags (3 SD, 1.5x IQR)
- Substantive flags (impossible values)
- Show flagged values in full-row context

### 4. Skip Patterns
- Identify filter variables and dependents
- Check values present when should be skipped
- Check missing when should be answered

### 5. Merge Diagnostics
- State expected cardinality (m:1, 1:1, m:m)
- Report match rates both directions
- Inspect unmatched observations
- Check for duplicated key inflation

### 6. Panel Consistency
- Balanced panel check
- Attrition analysis by key groups
- Time-invariant variables that vary

### 7. Treatment Variable
- Constant within unit across rounds
- Mutually exclusive and exhaustive
- Cross-tab with randomization strata
```

</details>

</details>

---

## Using These Skills

1. Install [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
2. Copy the skill file into `~/.claude/skills/<skill-name>/`
3. Start a Claude Code session — the skill triggers automatically when your task matches

Skills are plain text files. Edit them, add your own conventions, or use them as templates for new skills. If you build something useful, I'd love to hear about it — [email me](mailto:ejc93@georgetown.edu).
