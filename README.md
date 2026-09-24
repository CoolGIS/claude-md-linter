# claude-md-linter

Audit and slim down CLAUDE.md files against Anthropic's official "lean CLAUDE.md" best practices.

CLAUDE.md is loaded into context **every session**. Every line costs attention, and bloat buries the instructions that actually matter — Claude starts ignoring them. This skill lints for what should *not* be in the file, then proposes trimming changes as diffs. The bias is subtraction: only keep a line if removing it would cause Claude to make a concrete mistake.

## Install

```bash
npx skills add CoolGIS/claude-md-linter
```

Works with Claude Code, Cursor, Codex, Copilot, Windsurf, Gemini CLI and other agents supported by the [skills CLI](https://skills.sh/docs/cli).

## Usage

Just ask, in any project that has a CLAUDE.md:

- "Lint my CLAUDE.md — it's way too bloated"
- "Check whether my CLAUDE.md follows best practices"
- 「精简一下我的 CLAUDE.md，太臃肿了」

You get a structured report in the language you asked in (English or 中文): one block per finding with the violated rule, the reasoning behind it, and a diff preview — plus a summary with the estimated size reduction. **Nothing is edited until you explicitly approve which findings to apply** (e.g. "1, 3, 5" or "全部").

## What it checks

The core test applied to every line, section and example:

> If I removed this, would Claude make a mistake?

| Category | Example |
| --- | --- |
| `derivable-from-code` | file trees, per-file purpose blurbs |
| `standard-convention` | restating language/framework defaults |
| `docs-link-material` | verbose API docs that belong behind a link |
| `frequently-changing` | version lists, dependency tables |
| `tutorial-prose` | long how-it-works walkthroughs |
| `file-by-file-description` | directory-tree annotations |
| `self-evident-advice` | "write clean code" platitudes |
| `vague-instruction` | no verifiable criterion — gets tightened, not cut |

Gotchas ("here's where this project bites you") and non-obvious project-specific decisions are always preserved — that's the most valuable content a CLAUDE.md can hold.

## License

[MIT](./LICENSE)
