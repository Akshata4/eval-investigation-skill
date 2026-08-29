# Eval Investigation Skill

An autonomous, evidence-backed root-cause investigation methodology for AI-agent
evaluation failures. Given a low-scoring evaluator/metric, it drives an agent
to determine **what** failed, **how often**, **why**, **which system layer**
owns the behavior, and **whether the agent, tool, grader, or dataset is
actually responsible** — instead of assuming a low score means the agent is
wrong.

The whole skill is a single plain-text methodology
([`SKILL.md`](./SKILL.md)) with no framework dependency. It works with any
coding/agent harness that can be handed a system prompt, custom instructions,
or a slash-command prompt file — Claude Code, Codex CLI, Cursor, Windsurf,
Aider, a raw ChatGPT/Claude conversation, etc.

## Install

Pick your harness. Each command fetches the canonical file straight from
this repo into the path that harness auto-discovers.

### Claude Code

Project-scoped (one repo):

```bash
mkdir -p .claude/skills/eval-investigation
curl -fsSL https://raw.githubusercontent.com/Akshata4/eval-investigation-skill/main/SKILL.md \
  -o .claude/skills/eval-investigation/SKILL.md
```

User-scoped (every project):

```bash
mkdir -p ~/.claude/skills/eval-investigation
curl -fsSL https://raw.githubusercontent.com/Akshata4/eval-investigation-skill/main/SKILL.md \
  -o ~/.claude/skills/eval-investigation/SKILL.md
```

Then invoke it in a session with `/eval-investigation <what to investigate>`.

### Codex CLI

Codex reads custom prompts as markdown files under `.codex/prompts/`
(project) or `~/.codex/prompts/` (global), invoked with `/<filename>`:

```bash
mkdir -p .codex/prompts
curl -fsSL https://raw.githubusercontent.com/Akshata4/eval-investigation-skill/main/SKILL.md \
  -o .codex/prompts/eval-investigation.md
```

Then run `/eval-investigation <what to investigate>` in Codex CLI. The
leading YAML frontmatter (`name:`/`description:`) is inert to Codex — it's
only read by Claude Code — so no edits are needed.

### Any other harness (Cursor, Windsurf, Aider, plain system prompt, ...)

There's no universal auto-discovery convention across every tool, so the
reliable path is: fetch the file, then point your harness's
"custom instructions" / "rules" / "system prompt" mechanism at it (or paste
its body in).

```bash
curl -fsSL https://raw.githubusercontent.com/Akshata4/eval-investigation-skill/main/SKILL.md \
  -o eval-investigation.md
```

- **Cursor**: save as `.cursor/rules/eval-investigation.mdc`, or paste into
  a Project Rule.
- **Windsurf**: save as `.windsurfrules` (or append to it).
- **Aider / plain CLI agents**: pass with `--read eval-investigation.md`, or
  paste as the opening message.
- **A raw chat session**: paste the body of `SKILL.md` (everything after the
  `---` frontmatter block) as your first message, then follow it with what
  you want investigated.

## What it needs to be useful

This is a methodology, not a tool integration — it tells the agent *how* to
investigate, not *how to call your eval system*. It assumes the harness
already has some way to pull runtime evidence (traces, scores, feedback —
e.g. via a Langfuse MCP server or similar) and to read the connected source
repository. Without either, the agent will say so explicitly rather than
fabricate evidence (see "Evidence Standard" in `SKILL.md`).

## Repository layout

```
SKILL.md    canonical content — the single source of truth
README.md   this file
LICENSE     MIT
.claude/skills/eval-investigation/SKILL.md   symlink -> canonical (for this repo's own Claude Code use)
.codex/prompts/eval-investigation.md         symlink -> canonical (for this repo's own Codex CLI use)
```

Only `SKILL.md` is ever edited directly; the harness-specific paths are
symlinks so there's nothing to keep in sync.

## License

MIT — see [`LICENSE`](./LICENSE). Use it, fork it, adapt it for your own
eval system.
