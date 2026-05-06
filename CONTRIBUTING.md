# Contributing

## This Is a Personal Tools Repo

`paper-skills` is a set of [Cowork](https://www.anthropic.com/news/claude-skills) skills built around Alan Chester's academic-paper triage and Obsidian vault workflow. The defaults (theoretical pillars, tier tags, vault layout) reflect one specific research project. It is published openly so others can learn from it, fork it, and adapt it for their own use.

### What this means for contributions

- **You are welcome to fork this repo** and tailor the skills (pillars, tags, prompts, vault layout) to your own research workflow. That is the primary intended use case for anyone other than Alan.
- **PRs are welcome** for genuine bugs, broken skill prompts, or improvements that are broadly useful and not tied to one researcher's preferences.
- **Preference PRs will generally be declined.** If you prefer different tier semantics, a different annotation schema, or a different file-naming scheme, fork it. This repo is a specific person's research toolkit, not a general-purpose framework.
- **Alan has final say** on what goes into this repo.

If you are building your own paper-triage workflow, fork this repo and make it yours.
If you have found something that is broken or outdated in a way that affects everyone, open a PR.

---

## Branching, Commits, and Releases

The branching strategy, commit convention, and release process for this repo follow the canonical rules documented in the engineering handbook:

- **Why:** [Branching Strategy philosophy](https://github.com/amcheste/engineering-handbook/blob/main/docs/philosophies/branching-strategy.md)
- **How:** [Branching & Releases workflow](https://github.com/amcheste/engineering-handbook/blob/main/docs/workflows/branching-and-releases.md)

In short:

- Branch from `main`. One logical change per PR. PRs target `main`.
- [Conventional Commits](https://www.conventionalcommits.org/): `feat:` / `fix:` / `docs:` / `chore:` / `refactor:`, with `!` for breaking changes.
- Releases are tagged off `main` once the changes have settled.

Repo-specific contribution notes live below this line.

---

## Skill Authoring Notes

Each top-level folder (`paper-screen/`, `paper-annotate/`, `paper-intake/`) is one installable Cowork skill. Conventions to keep in mind when editing them:

- **Skill frontmatter must stay valid.** Every `SKILL.md` opens with a YAML block containing `name`, `description`, and any tool allowlist. The description is what surfaces to users when they pick the skill, so keep it action-oriented and specific.
- **Generic over personal.** The skills derive their pillars, tags, green/red flags, and coined terms from a user-supplied `.research-context.md` file in the vault root. Do not hardcode specific pillar names, journal lists, or vault paths into a skill; route them through `.research-context.md` instead.
- **Pipeline ordering matters.** `paper-intake` runs `paper-screen` first and halts on a SKIP verdict before `paper-annotate` runs. Changes to one skill's output schema can break the pipeline silently, so check that the screen output still parses cleanly into the annotate input.
- **Evals belong with skills.** When changing skill behavior, update or add a corresponding scenario in `evals/evals.json` so regressions get caught.
