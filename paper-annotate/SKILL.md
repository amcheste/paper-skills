# paper-annotate

Extract a structured annotation from an academic paper the user has already read. Produces a complete markdown file with YAML frontmatter and ten annotation fields, saved to the `Annotations/` folder in the user's Obsidian vault.

## Trigger

Use this skill when the user wants to annotate or take notes on a paper they have read. Triggers include:
- "annotate this paper"
- "create an annotation for..."
- "extract notes from this paper"
- "add this to my vault"
- "write up my notes on..."
- pasting a paper's full text or a large excerpt and asking for structured notes
- providing a PDF path and asking for an annotation

## Inputs

The user provides the paper content in one of these forms:

| Input form | Handling |
|---|---|
| Full text paste | Extract all fields directly from the text |
| PDF file path | Read the PDF, then extract |
| URL | Fetch the page, extract available content; note any fields that could not be determined from the page alone |
| Partial content (abstract + notes) | Extract what is available; mark remaining fields as `_to fill_` |
| Pre-filled notes | User pastes their own rough notes; reformat and complete using the schema below |

If the user provides their own notes or highlights alongside the paper content, incorporate them — especially into the **Research Notes** field and **Quotable Passage** (prefer user-marked quotes over your own selection).

## Research Context

Before annotating, load research context in this order:

1. **Check for `.research-context.md`** in the current working directory or vault root. If found, read it silently. Use its topic, pillars, and section structure to inform the **So What** and **Where to Cite** fields.
2. **If not found**, proceed with generic defaults. The So What field will ask the user to supply their research question, and Where to Cite will use generic section names (Introduction, Literature Review, Discussion, etc.).

## Output: YAML Frontmatter

Generate YAML frontmatter at the top of the annotation file with the following fields:

```yaml
---
title: "{Full paper title}"
authors: "{Last, F. I., & Last, F. I.}"
year: {YYYY}
journal: "{Journal or conference name}"
doi: "{DOI or URL if no DOI}"
date-annotated: {today's date, YYYY-MM-DD}
tags:
  - literature
  - tier/{1|2|3}
  - pillar/{pillar-slug}
---
```

### Tag Rules

**Tier tags** — assign exactly one based on verdict:

| Tag | Meaning |
|---|---|
| `#tier/1` | Core paper; directly supports a key argument in the user's work |
| `#tier/2` | Supporting paper; useful context or evidence, not load-bearing |
| `#tier/3` | Peripheral; worth knowing but unlikely to be cited heavily |

Tier is inferred from the So What assessment and the depth of connection to the research context. If uncertain, default to `#tier/2` and note the uncertainty in Research Notes.

**Pillar tags** — assign one or more from the following (use slug form):

| Tag | Pillar |
|---|---|
| `#pillar/org-design` | Org Design / Coordination Theory |
| `#pillar/multi-agent` | Multi-Agent Systems |
| `#pillar/future-of-work` | Future of Work / Human-AI Teaming |
| `#pillar/transaction-cost` | Transaction Cost Economics / Information Processing |

If `.research-context.md` defines different pillars, derive slugs by lowercasing and hyphenating the pillar names provided there (e.g., "Institutional Theory" → `#pillar/institutional-theory`).

## Output: Annotation Body

Produce the following ten fields in order, using the exact headings shown. Each heading is an H2.

---

### Field 1 — Core Claim

```markdown
## Core Claim
{One sentence. The single falsifiable claim the paper makes or defends. Not the topic — the argument.}
```

Aim for the form: *"{Subject} [does/shows/argues that] {claim}, [under condition / by mechanism]."*

---

### Field 2 — Mechanism

```markdown
## Mechanism
{2–4 sentences. How does the claim work? What is the causal or logical chain the authors propose?
This is the most important field. Capture the paper's internal logic, not just its conclusion.
If the paper proposes a model, framework, or typology, describe its structure here.}
```

Do not summarize the paper — explain the mechanism. If the paper has multiple mechanisms, pick the one most relevant to the research context and note the others briefly.

---

### Field 3 — Key Assumption

```markdown
## Key Assumption
{1–2 sentences. The most important hidden or stated assumption the claim rests on.
Choose the assumption whose failure would most directly invalidate the core claim.}
```

Avoid listing many assumptions. Pick one. If the paper explicitly states its assumptions, prefer one it treats as obvious or unquestioned — that is where hidden fragility lives.

---

### Field 4 — Quotable Passage

```markdown
## Quotable Passage
> "{Exact direct quote — do not paraphrase.}" (p. {page number})
```

Selection criteria, in priority order:
1. The single sentence or short passage that most precisely states the core claim or mechanism
2. A passage the user marked or highlighted (if provided)
3. A surprising or counterintuitive statement that makes the argument memorable

If working from a URL or abstract only (no page numbers available), omit the page reference and note: `(page unavailable — web source)`.

Do not fabricate quotes. If you cannot identify a confident direct quote, write:
```markdown
> _No direct quote extracted — full text not available. Suggested search term: "{key phrase from abstract}"_
```

---

### Field 5 — Limitations

```markdown
## Limitations
- {Limitation the authors acknowledge, stated as a brief phrase}
- {Second limitation}
- {Third limitation, if present}
```

Only list limitations the authors themselves acknowledge. Do not invent critiques. If the authors acknowledge none, write: `_Authors state no explicit limitations._` and optionally add one obvious methodological limitation in italics as your own observation, prefixed with `[Reviewer note:]`.

---

### Field 6 — So What

```markdown
## So What
{2–3 sentences. Why does this paper matter to the user's specific research?
If research context is loaded: connect the mechanism directly to the user's topic or argument.
If no research context: frame generically — what kind of argument or chapter does this paper equip?}
```

This field should be opinionated. Don't describe what the paper does — describe what the user can do with it. Use phrases like "This provides evidence for...", "This challenges the assumption that...", "This is the mechanism that explains..."

---

### Field 7 — Cross-References

```markdown
## Cross-References
- [[AuthorYear]] — {one phrase explaining the connection}
- [[AuthorYear]] — {one phrase explaining the connection}
```

Use Obsidian wikilink format `[[AuthorYear]]` where AuthorYear is the first author's last name concatenated with the four-digit year (e.g., `[[Brooks1975]]`, `[[Williamson1981]]`).

Sources to draw from, in priority order:
1. Papers the user has already mentioned or linked in conversation
2. Papers explicitly cited in the paper being annotated that are likely to be in the research context
3. Well-known foundational works in the relevant pillar(s)

List 2–5 cross-references. If none are identifiable, write `_None identified — add manually._`

---

### Field 8 — Where to Cite

```markdown
## Where to Cite
{Section name}: {1 sentence explaining what claim or point this paper supports in that section.}
```

If research context is loaded, use the actual section names from the user's paper outline or thesis structure.

If no context is loaded, use these generic section names: Introduction, Literature Review, Theoretical Framework, Methods, Findings / Analysis, Discussion, Conclusion.

A paper may appear in more than one section — if so, list each on its own line.

---

### Field 9 — Full Citation

```markdown
## Full Citation
{APA 7th edition citation, correctly formatted.}
```

APA 7th edition rules to follow:
- Authors: Last, F. I., & Last, F. I. (up to 20 authors; for 21+, list first 19, ellipsis, last author)
- Year in parentheses immediately after authors
- Title in sentence case (only proper nouns and first word after colon capitalized)
- Journal title in title case and italics (in markdown: *Journal Title*)
- Volume in italics, issue in parentheses not italicized
- Page range
- DOI as `https://doi.org/...` if available

If any field is missing, use a placeholder in angle brackets: `<volume>`, `<pages>`, `<doi>`.

---

### Field 10 — Research Notes

```markdown
## Research Notes
{Open field. Leave a placeholder if the user has not provided personal notes.}
```

If the user provided their own notes, thoughts, or marginalia alongside the paper content, paste and lightly format them here. Do not interpret or rewrite — preserve the user's voice.

If no user notes were provided, write:
```markdown
_Add your own notes here._
```

---

## Complete Output Template

The final file should look like this (with all fields populated):

```markdown
---
title: "{Title}"
authors: "{Authors}"
year: {YYYY}
journal: "{Journal}"
doi: "{DOI}"
date-annotated: {YYYY-MM-DD}
tags:
  - literature
  - tier/1
  - pillar/org-design
---

## Core Claim
...

## Mechanism
...

## Key Assumption
...

## Quotable Passage
> "..." (p. X)

## Limitations
- ...

## So What
...

## Cross-References
- [[AuthorYear]] — ...

## Where to Cite
Literature Review: ...

## Full Citation
...

## Research Notes
_Add your own notes here._
```

## File Naming and Save Location

Save the annotation file to `Annotations/` in the vault root (create the folder if it does not exist).

**Filename format:** `AuthorYear - Shortened Title.md`

Rules:
- AuthorYear: first author's last name + four-digit year (no space), e.g. `Brooks1975`
- Shortened Title: title-cased, up to 5 words, no punctuation except hyphens, e.g. `Mythical Man-Month`
- Full example: `Brooks1975 - Mythical Man-Month.md`

After writing the file, confirm the save path to the user:
```
Annotation saved to: Annotations/Brooks1975 - Mythical Man-Month.md
```

## Behavior Notes

- Never fabricate quotes, page numbers, DOIs, or citations. If a value cannot be determined, use a clearly marked placeholder.
- If the user provides only an abstract, complete all fields that are possible and mark the rest `_to fill_`. Do not refuse to annotate partial content.
- Mechanism is the most important field. Spend the most effort here. A vague mechanism entry is a failure.
- Keep Core Claim to one sentence. Resist the urge to hedge or qualify in that field — save nuance for Mechanism and Limitations.
- So What must connect to the user's actual work. A generic "this paper contributes to the field" is not acceptable.
- If the user asks to re-annotate or update an existing annotation, read the existing file first and preserve any content in Research Notes that the user has added.

## .research-context.md Format

See `paper-screen/SKILL.md` for the full `.research-context.md` specification. The `paper-annotate` skill reads the same file. Relevant fields for annotation:

```markdown
# Research Context

## Topic
{Used to calibrate So What and tier assignment.}

## Theoretical Pillars
{Used to assign #pillar/ tags and infer Where to Cite.}

## Paper Outline
{Optional. Section names and one-line descriptions. Used to populate Where to Cite with real section names.}
```

The `## Paper Outline` block is annotation-specific — `paper-screen` ignores it.
