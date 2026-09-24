---
name: claude-md-linter
description: Audit CLAUDE.md files for bloat and best-practice violations using Anthropic's official "lean CLAUDE.md" guidelines. This skill LINTS for what should NOT be in CLAUDE.md — content Claude can derive from code, standard language conventions, verbose API docs, frequently-changing info, long tutorials, file-by-file directory descriptions, and platitudes like "write clean code". Produces a structured report with each finding, the violated rule, the reasoning, and a diff preview, then ASKS the user before touching the file. Use this whenever the user wants to check, audit, slim down, trim, optimize, review, lint, or improve a CLAUDE.md, or says the CLAUDE.md is too long/bloated/verbose/noisy, or asks whether their CLAUDE.md follows best practices. Also triggers on Chinese phrasings："检查 CLAUDE.md", "精简 CLAUDE.md", "CLAUDE.md 最佳实践", "CLAUDE.md 太长了/太臃肿了", "瘦身 CLAUDE.md".
---

# CLAUDE.md Linter

Audit a CLAUDE.md file against Anthropic's official best-practice guidelines and propose **trimming** changes. The goal is a lean CLAUDE.md, because bloat dilutes the instructions Claude actually follows.

## Why this matters

CLAUDE.md is loaded into context **every single session**. Every line costs attention. When the file is bloated with things Claude already knows or could read from the code, the genuinely important instructions get buried — and Claude starts ignoring them. A short, dense CLAUDE.md that contains only what Claude would otherwise get wrong is far more effective than a comprehensive one.

The bias of this skill is **subtraction**. Default to cutting. Only keep something if removing it would cause Claude to make a concrete mistake.

## The core test

For every line, section, and example, ask:

> **"If I removed this, would Claude make a mistake?"**

- Honest answer "no" or "probably not" → cut it.
- "Yes, Claude would do the wrong thing without this" → keep it, and tighten the wording.

This is the single most important principle. Apply it relentlessly.

## Include / Exclude rules

| ✅ Keep | ❌ Cut |
| --- | --- |
| Bash commands Claude can't guess | Anything Claude can figure out by reading the code |
| Code style rules that differ from defaults | Standard language conventions Claude already knows |
| Testing instructions and preferred test runners | Detailed API documentation (link to docs instead) |
| Repository etiquette (branch naming, PR conventions) | Information that changes frequently |
| Architectural decisions specific to your project | Long explanations or tutorials |
| Developer environment quirks (required env vars) | File-by-file descriptions of the codebase |
| Common gotchas or non-obvious behaviors | Self-evident practices like "write clean code" |
| Concrete, verifiable instructions (commands, numbers, named standards) | Vague instructions Claude can't verify, like "Format code properly" or "Test your changes" |

Each ❌ row maps to a concrete lint category. Use these names in the report:

1. **derivable-from-code** — restates structure, types, or behavior visible in the repo (file trees, per-file purpose blurbs, routine config explanations).
2. **standard-convention** — restates a language/framework default Claude already follows.
3. **docs-link-material** — verbose API/library docs that belong behind a hyperlink.
4. **frequently-changing** — version numbers, dependency lists, file counts, module-status tables that go stale on every change.
5. **tutorial-prose** — long explanations, walkthroughs, how-it-works narrative.
6. **file-by-file-description** — directory-tree annotations describing what each file does.
7. **self-evident-advice** — platitudes like "write clean code", "handle errors gracefully", "use meaningful names".
8. **vague-instruction** — gives no verifiable criterion: no command, number, or named standard ("Format code properly" instead of "Use 2-space indentation"). The remedy is TIGHTEN into a concrete instruction, not CUT.

## Workflow

### 1. Locate the target

- If the user named a path, use it.
- Otherwise default to `./CLAUDE.md` in the current project. Offer to also check `~/.claude/CLAUDE.md` (global) and nested `CLAUDE.md` files if relevant, but lint one file per report unless the user asks for several.
- Read the full file before judging anything. Surface-level skimming produces wrong calls on content that looks verbose but encodes a real gotcha.

### 2. Classify each section

Walk the file top to bottom. For every heading, paragraph, code block, and list item, decide:

- **KEEP** — passes the core test (removing it would cause a mistake).
- **CUT** — matches one of the 7 bloat categories above.
- **TIGHTEN** — the idea is worth keeping but the wording is verbose, or it's a vague-instruction (category 8); rewrite shorter and more concrete.

Be honest. Polite restraint hurts here. A section the author worked hard on is not automatically worth keeping — sunk cost is not a reason to keep bloat.

### 3. Check for contradictions

After classifying, reread the whole file once looking only for rules that contradict each other — official guidance: "if two rules contradict each other, Claude may pick one arbitrarily". Contradictions often hide across distant sections (a style rule vs. a legacy-code exception, two different test commands). Report each as its own finding with the category name **contradiction**; the remedy is reconciling or deleting one side, and the user decides which.

### 4. Produce the report

Output the report in **the same language the user is speaking** (中文提问 → 中文报告, English → English). Technical identifiers stay in their original form.

Use one block per finding, in document order. Follow this exact structure:

```
## 发现 N — <类别名>

**位置：** <标题路径或行号范围，如 "## 技术栈 > Vue 3.5" 或 "L12-18">

**问题：** <一两句话说明哪里违规>

**为什么有害：** <具体后果——Claude 可能犯什么错，或"稀释了高信号指令，让真正重要的内容被忽略">

```diff
- <建议删除的原文行；或->
+ <TIGHTEN 时写更精简的替换；纯 CUT 时这里留空或不出现 + 行>
```
```

Merge adjacent CUTs of the same category into one finding — don't pad the report with twenty near-identical entries.

### 5. Summary

End the report with a short summary:

```
## 总结

- 发现数：N（X 处 CUT，Y 处 TIGHTEN，Z 处保留）
- 预估精简：约 XX%（从 NNN 行 → 约 MMM 行）
- 最高价值改动：<哪一条，以及为什么>
```

### 6. Ask, then wait

After the report, ask the user which findings to apply. **Do not edit CLAUDE.md yet.** Suggested phrasing:

> 以上是 N 处建议。告诉我你想应用哪些（比如 "1, 3, 5" 或 "全部"），我再动手改；不想改的也可以直接说，或对任何一条提出异议。

Only edit after the user gives explicit approval — specific numbers, "all", "apply them", "全部", "都改" all count. If the reply is ambiguous or asks a follow-up question, clarify before editing. **Never assume consent.** Silence is not consent; if the user changes subject without approving, do not edit.

### 7. Apply approved changes

When applying:

- Re-read the file before editing in case anything changed since the report.
- Apply only the approved findings. Skip anything the user didn't confirm — partial approval is the norm, not the exception.
- Use whatever editing tool the project conventions call for (plain markdown → Edit; code files → per project rules).
- After editing, report what changed and the new line count. Don't re-run a full audit unless asked.

## Judgment guidance

- **Be concrete about the downside.** "It's verbose" is weak reasoning. "A reader scanning for the deploy command lands on this paragraph and gives up" is strong. The report should make the cost of each line tangible.
- **Respect project-specific knowledge.** If something looks like a standard convention but the project has a documented reason to deviate, keep it — that's exactly the kind of non-obvious decision that belongs in CLAUDE.md. When in doubt, flag it as "建议向用户确认" rather than cutting.
- **Don't invent rules.** Only the 8 categories above plus the whole-file contradiction check. If something is mildly suboptimal but doesn't fit, leave it alone — over-linting erodes trust and makes the user dismiss real findings.
- **Concise ≠ vague.** A short line that names a command, a number, or a concrete standard passes vague-instruction; flag only instructions with no verifiable criterion at all.
- **Keep gotchas at all costs.** The most valuable CLAUDE.md content is "here's where this project bites you." Never cut a gotcha even if it's long; tighten the wording instead. A 5-line gotcha that prevents one debugging session is worth more than a 50-line architecture overview.
- **Prefer a link over deletion when useful.** When cutting docs-material, suggest replacing with a one-line pointer (`API: see <url>`) rather than deleting outright, where a link genuinely helps.
- **Version pins cut both ways.** A specific Node version requirement (`^20.19.0`) is environment-quirk → keep. A dependency list with version numbers that updates every release is frequently-changing → cut.
