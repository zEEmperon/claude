# Claude Skill Manager

This repo is a Claude Code skill manager. The meta-skills under `.claude/skills/` auto-load when this repo is open in Claude Code.

## Layout

```
.claude/skills/<meta-skill>/SKILL.md   auto-loaded
.claude/skills/_shared/CONVENTIONS.md  shared reference (read on demand)
skills/<skill-name>/SKILL.md           local skill feed (gitignored, flat)
```

`skills/` is the user's local staging area. Each skill is a flat folder; the folder name must equal the `name` field in SKILL.md frontmatter.

## When working in this repo

- Don't reintroduce categories under `skills/`. Skills are flat, matching how Claude Code stores them in `.claude/skills/`.
- Don't maintain per-skill tables in README.md or CLAUDE.md. `skills/` is gitignored, so any list will diverge from reality.
- For meta-skills, prefer Claude's native tools (Read, Write, Glob, Edit) over shell. Use Bash only when shell is genuinely needed; PowerShell only as a Bash fallback. Do not triplicate commands for Linux/macOS/Windows — Bash via git-bash works on all three.
- Read `.claude/skills/_shared/CONVENTIONS.md` for scope paths, the manifest schema, and tool guidance instead of restating them in each skill.
