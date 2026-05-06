# paper-skills

Cowork skills for academic research paper processing. Triage papers, extract structured annotations, and file everything directly into your Obsidian vault, from a single command.

---

## Skills

### `paper-screen`
Triage a paper before you commit to reading it. Paste a title and abstract (or drop a URL) and get back a **READ / MAYBE / SKIP** verdict with a 0–100 relevance score, a theoretical pillar classification, and a 2–3 sentence rationale. Supports batch mode: paste a list of papers and get a full triage table.

### `paper-annotate`
Extract a structured 10-field annotation from a paper you've already read. Produces a markdown file with YAML frontmatter and Obsidian wikilinks, ready to drop into your vault. Fields: Core Claim, Mechanism, Key Assumption, Quotable Passage, Limitations, So What, Cross-References, Where to Cite, Full Citation (APA 7th), and Research Notes.

### `paper-intake`
The full pipeline in one command. Runs `paper-screen` → `paper-annotate` → terminology check → save to `Annotations/`. If a paper screens as SKIP, the pipeline halts and tells you why. No annotation is created. Ends with a completion report: verdict, score, tier, pillar, best use in your paper, and three suggested next actions.

---

## How They Fit Together

```
Input (URL / paste / PDF)
        │
        ▼
  paper-screen ──► SKIP? Stop here.
        │
        ▼ READ or MAYBE
  paper-annotate (10 fields + YAML frontmatter)
        │
        ▼
  Terminology check (coined terms vs. paper text)
        │
        ▼
  Save to Annotations/ ──► Completion report
```

Run the skills individually when you only need one step, or use `paper-intake` to run the whole pipeline at once.

---

## Obsidian Integration

Annotations are saved as markdown files to `Annotations/` in your vault root, named `AuthorYear - Shortened Title.md`. Each file includes:

- **YAML frontmatter** with title, authors, year, journal, DOI, date annotated, tier tags (`#tier/1`, `#tier/2`, `#tier/3`), and pillar tags (`#pillar/org-design`, `#pillar/multi-agent`, etc.)
- **Obsidian wikilinks** in the Cross-References field (`[[Brooks1975]]`, `[[Williamson1981]]`) for graph view connections
- **Callout block formatting** on screen output, for direct paste into any note

The skills are generic. They derive tags and pillar classifications from your `.research-context.md` config, so the vault structure works for any research project.

---

## Research Context

All three skills read a single config file (`.research-context.md`) from your vault root. This file tells the skills what your research is about, what counts as relevant, and what terms are uniquely yours.

If the file doesn't exist, the skills fall back to defaults and ask one question before proceeding.

### Example `.research-context.md`

```markdown
# Research Context

## Topic
How do coordination costs scale in multi-agent AI systems, and what organizational
design principles from human teams transfer to AI orchestration architectures?

## Theoretical Pillars
1. Org Design / Coordination Theory — classical and contemporary theory on how
   teams structure work and manage dependencies
2. Multi-Agent Systems — architectures, protocols, and emergent behavior in
   collections of AI agents
3. Future of Work / Human-AI Teaming — how humans and AI systems divide labor,
   share context, and maintain trust
4. Transaction Cost Economics / Information Processing — Williamson, Simon, and
   the economics of coordination and governance

## Green Flags
coordination costs, coordination overhead, multi-agent, human-AI, orchestration,
Brooks' Law, team topology, communication overhead, bounded rationality,
transaction cost, span of control, modular organization, collective intelligence,
cognitive load, autonomous agents, agent coordination

## Red Flags
hardware design, medical imaging, clinical trial, drug discovery, computer vision,
financial forecasting, game engine, genomics

## Coined Terms
coordination debt, agent span of control, delegation bandwidth, orchestration overhead

## Paper Outline
1. Introduction — motivation and research question
2. Literature Review — coordination theory, MAS, and human-AI teaming
3. Theoretical Framework — the coordination cost model
4. Analysis — applying the framework to AI orchestration cases
5. Discussion — design principles and implications
6. Conclusion
```

All fields are optional. Any omitted field falls back to the skill's built-in defaults.

---

## Quick Start

**1. Install the skills**

Package each skill folder as a `.skill` file and install via Cowork in the Claude desktop app. Each folder (`paper-screen/`, `paper-annotate/`, `paper-intake/`) corresponds to one installable skill.

**2. Create your research context** *(optional but recommended)*

Create `.research-context.md` in your Obsidian vault root using the template above. Fill in your topic, pillars, green/red flags, coined terms, and paper outline. The skills read this file automatically. No other configuration needed.

**3. Process a paper**

Drop a URL or paste a title and abstract, then run the intake skill:

> *"Run paper-intake on this paper: [URL or paste]"*

Or use the skills individually:

> *"Screen this paper: [title + abstract]"*
> *"Annotate this paper: [paste or PDF path]"*

**4. Find your annotations**

Completed annotations appear in `Annotations/` in your vault root, named `AuthorYear - Shortened Title.md`. Open your vault in Obsidian. Wikilinks and tags are already in place.

---

## Skill Reference

| Skill | Input | Output | Stops early if... |
|---|---|---|---|
| `paper-screen` | Title + abstract, URL, or list | Verdict callout block or triage table | — |
| `paper-annotate` | Full text, PDF, URL, or partial content | Markdown file in `Annotations/` | — |
| `paper-intake` | Title + abstract minimum, URL preferred | Completion report + file in `Annotations/` | Verdict is SKIP |

**Tier tags:**
- `#tier/1`: core paper; directly supports a key argument
- `#tier/2`: supporting context; useful but not load-bearing
- `#tier/3`: peripheral; worth knowing, unlikely to be cited heavily

**Default pillar tags** (overridden by `.research-context.md`):
- `#pillar/org-design`
- `#pillar/multi-agent`
- `#pillar/future-of-work`
- `#pillar/transaction-cost`

---

## Related

**[ea-skills](https://github.com/amcheste/ea-skills)**: Cowork skills for enterprise architecture work.

---

## License

[MIT](LICENSE)
