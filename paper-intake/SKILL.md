# paper-intake

Full pipeline orchestrator. Runs `paper-screen` → `paper-annotate` → terminology check → save → completion report in sequence. The single command to process a paper from cold input to filed annotation.

## Trigger

Use this skill when the user wants to fully process a paper in one pass. Triggers include:
- "intake this paper"
- "process this paper"
- "run the full pipeline on..."
- "add this paper to my vault"
- "screen and annotate this"
- providing a URL or PDF with no other instruction in a research context
- "run intake on these papers" (batch — see Batch Mode below)

Do not use this skill if the user asks only to screen (use `paper-screen`) or only to annotate (use `paper-annotate`).

## Inputs

Accept any of the following as the starting point:

| Input form | Handling |
|---|---|
| URL | Fetch and parse. Extract title, abstract, authors, year, venue, full text if available |
| PDF file path | Read the PDF, extract all fields |
| Title + abstract (minimum) | Proceed directly to Step 1 |
| Title only | Proceed with reduced confidence; note missing abstract in output |
| Paste of full text | Extract metadata from the text, proceed |
| Batch list | See Batch Mode section |

## Research Context

Before starting the pipeline, load `.research-context.md` from the vault root / current working directory if it exists. Read it once and carry its values through all five steps — do not re-read it at each step.

Fields used per step:

| Field | Used in |
|---|---|
| `## Topic` | Step 1 (scoring), Step 2 (So What, tier), Step 5 (next actions) |
| `## Theoretical Pillars` | Step 1 (pillar classification), Step 2 (pillar tags) |
| `## Green Flags` | Step 1 (score boost) |
| `## Red Flags` | Step 1 (score penalty) |
| `## Coined Terms` | Step 3 (terminology check) |
| `## Paper Outline` | Step 2 (Where to Cite), Step 5 (next actions) |

If `.research-context.md` is not found:
- Ask the user one question before starting: *"What is your research topic or question, and do you have any key terms or coined phrases I should check for?"*
- Accept a brief answer; extract topic and coined terms from it. Do not ask follow-up questions — proceed.

---

## Step 1 — Screen

Run the `paper-screen` skill on the input.

Produce the screen output as a collapsed block so it is available but does not dominate the response:

```markdown
<details>
<summary>Screen result — {Title} ({Score}/100, {VERDICT})</summary>

{Full paper-screen output callout block here}

</details>
```

### Gate condition

| Verdict | Action |
|---|---|
| **SKIP** | Stop the pipeline. Output the Stop Report (see below). Do not proceed to Step 2. |
| **MAYBE** | Continue with a caution note: *"Verdict is MAYBE — annotating, but tier will be capped at tier/2."* |
| **READ** | Continue normally. |

#### Stop Report (SKIP only)

When the pipeline halts at Step 1, output:

```markdown
## Pipeline halted — SKIP

**Paper:** {Title}
**Score:** {score}/100
**Reason:** {2–3 sentence rationale from paper-screen}
**Matched red flags:** {list}

No annotation was created. To override and annotate anyway, run: `paper-annotate` directly.
```

Then stop. Do not execute Steps 2–5.

---

## Step 2 — Annotate

Run the `paper-annotate` skill on the same input.

Use all ten fields as specified in `paper-annotate/SKILL.md`. Do not re-ask questions already answered in Step 1 — carry over: title, authors, year, journal/venue, pillar classification, and relevance signals.

If the verdict from Step 1 was MAYBE, cap the tier tag at `#tier/2` regardless of what the So What assessment might suggest. Note this in the Research Notes field:
```
[paper-intake] Tier capped at 2 — screen verdict was MAYBE.
```

Do not output the full annotation body inline here. Save it directly to file in Step 4. In the pipeline progress output, show only:

```
✓ Step 2 complete — annotation drafted (10/10 fields)
  Tier: {tier}  |  Pillar: {pillar}  |  Where to cite: {section}
```

If any fields could not be completed (e.g., abstract-only input), show:
```
✓ Step 2 complete — annotation drafted ({n}/10 fields; {list incomplete fields})
```

---

## Step 3 — Terminology Check

Check whether any of the user's coined terms or key phrases appear in the paper.

**Source of terms**, in priority order:
1. `## Coined Terms` block in `.research-context.md`
2. Terms provided by the user in response to the pre-pipeline question
3. If neither is available, skip this step and note: `⚠ Step 3 skipped — no coined terms defined.`

### Check procedure

For each coined term:
- Search the paper text (case-insensitive, partial match acceptable) for the term or close variants
- Report whether it appears, and if so, quote the sentence(s) where it appears with page/location if available

### Output format

```markdown
## Terminology Check

| Term | Found | Location / Quote |
|---|---|---|
| {coined term} | Yes | "...{sentence containing term}..." (p. {n}) |
| {coined term} | No | — |
| {coined term} | Partial | "{closest phrase found}" — not exact match |
```

After the table, add a one-sentence interpretation:
- If any terms are found: *"This paper uses [{term}] — potential alignment with your framework or prior art to address."*
- If none found: *"None of your coined terms appear in this paper — it is unlikely to compete with or duplicate your contribution."*
- If partial matches: *"The paper uses [{closest phrase}] where you use [{coined term}] — worth checking whether these concepts overlap."*

The terminology check does not affect the score, tier, or verdict. It is informational only.

---

## Step 4 — Save

Write the completed annotation to file.

**Save location:** `Annotations/` folder in the vault root. Create the folder if it does not exist.

**Filename:** `AuthorYear - Shortened Title.md` (same convention as `paper-annotate` — first author last name + year, no space; title-cased shortened title up to 5 words, hyphens only).

**File contents:** Full annotation as specified in `paper-annotate/SKILL.md` — YAML frontmatter followed by all ten fields.

Append a pipeline metadata block at the bottom of the file, after the ten annotation fields:

```yaml
---
pipeline:
  run-by: paper-intake
  date: {YYYY-MM-DD}
  screen-score: {score}
  screen-verdict: {READ|MAYBE}
  terminology-hits: {n of m terms found}
---
```

After writing the file, confirm:
```
✓ Step 4 complete — saved to Annotations/{filename}.md
```

If a file with the same name already exists, do not overwrite silently. Report:
```
⚠ File already exists: Annotations/{filename}.md
  Options: (a) overwrite, (b) save as {filename}-2.md, (c) cancel
```
Wait for the user to choose before writing.

---

## Step 5 — Completion Report

Output a final pipeline summary. This is the primary output the user sees — Steps 1–4 are collapsed or brief.

```markdown
---

## Intake Complete — {Title}

| Field | Value |
|---|---|
| **Verdict** | {READ / MAYBE} |
| **Relevance score** | {score}/100 |
| **Tier** | {tier/1 \| tier/2 \| tier/3} |
| **Pillar** | {Pillar name} |
| **Best use** | {The single most valuable way to use this paper in the user's work — one sentence, specific} |
| **Annotation saved** | `Annotations/{filename}.md` |
| **Terminology hits** | {n}/{total} coined terms found {— or — (step skipped)} |

### Suggested Next Actions

1. {Specific, actionable next step — e.g., "Read §3 (Mechanism) closely — the coordination cost model maps directly to your Chapter 2 argument."}
2. {Second next step — e.g., "Follow the citation trail: [[SimonYear]] is cited three times and may be a stronger primary source for the bounded rationality claim."}
3. {Third next step — e.g., "Update your literature map — this paper bridges [[AuthorYear]] and [[AuthorYear]] and should sit between them."}

---
```

### Next Actions — generation rules

Next actions must be:
- **Specific to this paper and this user's research** — not generic advice like "read it carefully"
- **Actionable** — the user should be able to do them immediately or add them to a task list
- Drawn from: the mechanism, the cross-references, the Where to Cite field, and the terminology check results

Draw from these categories (pick the three most useful):
- A specific section of the paper to read or re-read
- A citation to follow from the paper's bibliography
- A gap or assumption to explore (from Limitations or Key Assumption fields)
- An update to make in the vault (literature map, outline, another annotation)
- A terminology conflict or alignment to resolve (from Step 3)
- A claim in the paper that challenges or supports a specific argument in the user's paper

Do not pad with filler actions. If fewer than three meaningful actions are identifiable, provide two strong ones rather than three weak ones.

---

## Batch Mode

When the user provides a list of papers (titles, abstracts, URLs), run the full pipeline on each in sequence.

**Batch order:** Sort by screen score descending — process the highest-scoring papers first.

**Batch output structure:**

First, run Step 1 on all papers and produce a triage table:

```markdown
## Batch Triage

| # | Title | Score | Verdict | Pillar |
|---|---|---|---|---|
| 1 | {Title} | {score}/100 | READ | {Pillar} |
| 2 | {Title} | {score}/100 | MAYBE | {Pillar} |
| 3 | {Title} | {score}/100 | SKIP | — |
```

Confirm with the user before proceeding:
*"Found {n} READ, {m} MAYBE, {k} SKIP. Proceed with full intake on READ and MAYBE papers? SKIP papers will not be annotated."*

Wait for confirmation, then run Steps 2–5 for each non-SKIP paper in score order. Output a mini completion report per paper:

```markdown
### {Title}
- Tier: {tier} | Pillar: {pillar} | Score: {score}/100
- Saved: `Annotations/{filename}.md`
- Terminology hits: {n}/{m}
- Best use: {one sentence}
```

After all papers are processed, output a vault summary:
```markdown
## Batch Complete
- {n} annotations saved to Annotations/
- {m} papers skipped
- {k} terminology hits across {j} papers
```

---

## Pipeline Progress Display

During execution, show a live progress indicator between steps:

```
[paper-intake] Starting pipeline for: {Title}

  Step 1 ▸ Screening...
  ✓ Step 1 complete — {VERDICT} ({score}/100)

  Step 2 ▸ Annotating...
  ✓ Step 2 complete — annotation drafted ({n}/10 fields)
    Tier: {tier}  |  Pillar: {pillar}  |  Where to cite: {section}

  Step 3 ▸ Terminology check...
  ✓ Step 3 complete — {n}/{m} terms found

  Step 4 ▸ Saving...
  ✓ Step 4 complete — saved to Annotations/{filename}.md

  Step 5 ▸ Building completion report...
```

This keeps the user oriented during a multi-step run without requiring them to read intermediate output in detail.

---

## Behavior Notes

- **Do not re-ask for information already provided.** If the user gave a URL in the intake command, do not ask for the title separately.
- **Carry state across steps.** The pillar assigned in Step 1 is the pillar used in Step 2 tags. The tier from Step 2 is reported in Step 5. Nothing is re-derived independently.
- **SKIP is a hard gate.** Do not proceed to annotation for a SKIP paper, even if the user seems interested. Surface the Stop Report and let them invoke `paper-annotate` directly if they want to override.
- **MAYBE papers get tier/2 at most.** This is a firm rule, not a suggestion.
- **Steps 2 and 3 share the same paper text.** Do not re-fetch or re-read the paper between steps.
- **Step 5 is the deliverable.** The user's primary experience is the completion report. Steps 1–4 are infrastructure.
- **Do not hallucinate.** No fabricated quotes (Step 2), no invented terminology matches (Step 3), no invented file paths (Step 4).
- **File conflicts require user confirmation.** Never silently overwrite an existing annotation.

## .research-context.md Format

`paper-intake` reads the same `.research-context.md` as `paper-screen` and `paper-annotate`, with one additional optional block:

```markdown
# Research Context

## Topic
{Research question or focus.}

## Theoretical Pillars
1. {Pillar name} — {description}
2. ...

## Green Flags
{Signal words that boost relevance score.}

## Red Flags
{Signal words that lower relevance score.}

## Coined Terms
{Newline or comma-separated list of your original terms, phrases, or framework names to check for in each paper.}

## Paper Outline
{Section names and one-line descriptions — used to populate Where to Cite.}
```

The `## Coined Terms` block is intake-specific. `paper-screen` and `paper-annotate` ignore it.
