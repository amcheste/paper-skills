# paper-screen

Screen an academic paper for relevance before reading it. Returns a READ / MAYBE / SKIP verdict with a numeric score, theoretical pillar classification, and a brief rationale formatted for Obsidian.

## Trigger

Use this skill when the user wants to evaluate whether a paper is worth reading. Triggers include phrases like:
- "screen this paper"
- "should I read this?"
- "is this paper relevant?"
- "triage these papers"
- "score this abstract"
- pasting a URL to a paper page (arXiv, ACM DL, Semantic Scholar, SSRN, etc.)
- pasting a list of paper titles/abstracts

## Inputs

Accept any combination of the following. If given a URL, fetch the page and extract the fields automatically.

| Field | Required | Notes |
|---|---|---|
| Title | Yes (or URL) | Full paper title |
| Abstract | Strongly preferred | Core signal for scoring |
| Authors | Optional | Noted in output |
| Year | Optional | Noted in output |
| Journal / Venue | Optional | Noted in output |
| URL | Alternative to above | Fetch and parse automatically |

For **batch mode**: the user pastes a newline-separated list of papers (titles, or title + abstract blocks, or a BibTeX snippet). Screen each one and return a single triage table.

## Research Context

Before screening, load research context in this order:

1. **Check for `.research-context.md`** at the vault root (i.e., look for a file named `.research-context.md` in the current working directory or the directory of the file being worked on). If found, read it silently and use its definitions for topic, pillars, green flags, and red flags.
2. **If not found**, ask the user one question before proceeding:
   > "What is your research topic or question? (I'll use this to calibrate the relevance score.)"
   Then use the default pillars and signal words below, supplemented by anything the user provides.

### Default Research Context (fallback)

**Topic:** Not specified — use general academic relevance heuristics.

**Theoretical Pillars** (classify paper into the single best fit):
1. Org Design / Coordination Theory
2. Multi-Agent Systems
3. Future of Work / Human-AI Teaming
4. Transaction Cost Economics / Information Processing

**Green-flag signal words** (each match boosts score):
- coordination costs, coordination overhead
- multi-agent, multi-agent systems, MAS
- human-AI, human-AI teaming, human-in-the-loop
- orchestration, conductor, delegation
- Brooks' Law, mythical man-month
- team topology, team structure, team boundaries
- communication overhead, communication cost
- information processing, bounded rationality
- transaction cost, governance cost
- organizational design, org design
- modularity, modular organization
- collective intelligence, swarm intelligence
- cognitive load, cognitive overhead
- span of control, managerial span
- autonomous agents, agent coordination

**Red-flag signal words** (each match lowers score):
- hardware design, circuit, VLSI, embedded systems
- medical imaging, radiology, clinical trial, patient outcomes
- natural language processing (as primary topic unrelated to coordination)
- computer vision (as primary topic unrelated to coordination)
- drug discovery, genomics, protein folding
- financial forecasting, stock prediction, trading strategy
- game engine, graphics rendering, simulation physics
- unrelated domain (agriculture, geology, meteorology, etc.)

## Scoring Algorithm

Compute a relevance score from 0–100 using this rubric:

| Factor | Weight | Notes |
|---|---|---|
| Abstract keyword match (green flags) | +5 per hit, max +40 | Case-insensitive, partial match ok |
| Title keyword match (green flags) | +5 per hit, max +20 | Higher signal than abstract alone |
| Red flag presence | -10 per hit, min 0 | Hard domain mismatch |
| Venue relevance | +0 to +10 | Top venues in the pillar area get full +10 |
| Recency (2018–present) | +5 | Older papers still ok, just no bonus |
| Foundational / canonical work | +10 | Seminal papers get a bonus regardless of year |
| User-supplied topic alignment | +0 to +15 | Subjective; use judgment against stated topic |

**Verdict thresholds:**

| Score | Verdict |
|---|---|
| 70–100 | **READ** |
| 40–69 | **MAYBE** |
| 0–39 | **SKIP** |

## Output Format

### Single Paper

Produce output as a fenced Obsidian callout block, ready to paste into a note.

```
> [!note] Paper Screen — {Title}
> **Verdict:** READ / MAYBE / SKIP — Score: {0-100}/100
> **Pillar:** {Pillar name}
> **Authors:** {Authors} ({Year}) — {Journal/Venue}
> **Rationale:** {2–3 sentences explaining the verdict. Reference specific concepts from the abstract that matched or missed. Be direct.}
> **Matched signals:** {comma-separated green flags found}
> **Red flags:** {comma-separated red flags found, or "none"}
> **URL:** {url if provided}
```

Example:

```
> [!note] Paper Screen — The Mythical Man-Month Revisited
> **Verdict:** READ — Score: 88/100
> **Pillar:** Org Design / Coordination Theory
> **Authors:** F. Brooks (1995) — Addison-Wesley
> **Rationale:** Directly addresses coordination overhead and communication costs in software teams, both core green-flag concepts. The foundational treatment of Brooks' Law makes this canonical reading for any coordination-cost research. No red flags present.
> **Matched signals:** coordination costs, communication overhead, Brooks' Law, team structure
> **Red flags:** none
> **URL:** —
```

### Batch Mode

Return a Markdown table followed by individual rationale entries for any paper scored READ or MAYBE.

**Triage table:**

```markdown
| # | Title | Score | Verdict | Pillar | Key Signals |
|---|---|---|---|---|---|
| 1 | {Title} | {score}/100 | READ | {Pillar} | {top 2-3 signals} |
| 2 | {Title} | {score}/100 | MAYBE | {Pillar} | {top 2-3 signals} |
| 3 | {Title} | {score}/100 | SKIP | — | {red flags if any} |
```

Then, for each READ or MAYBE paper, append the full single-paper callout block.

SKIP papers get only the table row — no callout.

## Behavior Notes

- If the user provides only a title with no abstract, note the uncertainty and widen the score range (prefer MAYBE over confident READ/SKIP unless the title is unambiguous).
- If a URL is provided, fetch the page. Extract title, authors, year, venue, and abstract. If extraction fails, tell the user which fields are missing and ask them to paste the abstract.
- Do not hallucinate paper content. If you are uncertain about what a paper argues, say so in the rationale.
- In batch mode, process all papers before outputting — emit the full table at once, not incrementally paper-by-paper.
- Keep rationale tight: 2–3 sentences maximum. No hedging filler. Be opinionated.

## .research-context.md Format

If the user wants to customize this skill for their project, they can create `.research-context.md` in their vault root with the following structure:

```markdown
# Research Context

## Topic
{One paragraph describing the research question or project focus.}

## Theoretical Pillars
1. {Pillar 1 name} — {brief description}
2. {Pillar 2 name} — {brief description}
3. {Pillar 3 name} — {brief description}
4. {Pillar 4 name} — {brief description}

## Green Flags
{comma or newline separated list of signal words/phrases that indicate relevance}

## Red Flags
{comma or newline separated list of signal words/phrases that indicate irrelevance}
```

All fields are optional; any omitted field falls back to the defaults defined in this skill.
