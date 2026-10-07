# SanityOps Inspect Lite — Quick Start

A lightweight inspector for AI Agent logic artifacts (**System Prompt / Skill / Tool Schema**): **report-only** (detect-only), built on SanityOps Framework v1.0.

## Option 1: Claude Code users (recommended)

```
/plugin marketplace add sanityops-org/sanityops-inspect-lite
/plugin install sanityops-inspect-lite@sanityops
```

Then just say "inspect my system prompt / skill / tool schema" to trigger it.

## Option 2: Clone / download

```
git clone https://github.com/sanityops-org/sanityops-inspect-lite.git
```

The skill lives in `plugins/sanityops-inspect-lite/skills/sanityops-inspect-lite/` (`SKILL.md` + 6 checklists in `references/`). Install that folder as a skill in your agent.

## Data & privacy

- The skill itself is **pure Markdown**: no scripts, no dependencies, no network requests.
- **Detect-only**: it reads and reports; it never modifies your artifacts.
- Inspection runs in your local **host model**; whether content leaves your environment depends on the model you use (for example, Claude Code's cloud model receives your content).

## Boundaries

- ✅ Five-stage defect inspection: single-artifact → cross-artifact → Gate-0 → permission → impact outlook
- ✅ Reports only; never rewrites any artifact
- ❌ Does not run the Risk / Quality subsets; for full capability use the official `sanityops-cli`
