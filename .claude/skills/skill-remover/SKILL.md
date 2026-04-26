---
name: skill-remover
description: >
  Remove a previously installed skill from a Claude Code project or from the global Claude Code config.
  Use when a user wants to uninstall, remove, delete, or clean up a skill.
  Triggers for: "remove skill", "uninstall skill", "delete skill", "clean up skill".
---

# Skill Remover

Deletes an installed skill directory. See `.claude/skills/_shared/CONVENTIONS.md` for scope paths and tool-choice rules.

## Step 1 — Extract intent

Identify:
- **Skill name**
- **Scope** — `local`, `workspace`, or `global`

## Step 2 — Fill in missing info (one message)

- If scope is missing: ask "global, local (this project), or a specific workspace path?".
- If scope is `workspace` and no path: ask for an absolute path.
- If skill name is missing: Glob `<skills_path>/*/SKILL.md` for the chosen scope, list names, ask which to remove.
- If both are known: skip listing.

## Step 3 — Resolve and verify

`<SKILLS_PATH>`:
- `local` → `.claude/skills` (cwd)
- `workspace` → `<user-path>/.claude/skills`
- `global` → `~/.claude/skills`

`<TARGET>` = `<SKILLS_PATH>/<skill-name>`. Verify it exists; if not, report and stop.

## Step 4 — Confirm before deleting

Show:

```
About to delete: <TARGET>
Proceed? (yes/no)
```

Do not delete without explicit "yes".

## Step 5 — Delete

```bash
rm -rf "<TARGET>"
```

## Step 6 — Confirm

Report skill name, scope, deleted path, and: "If Claude Code is running, the skill is no longer available."

## Multiple skills

Confirm per skill, then report once.
