---
name: skill-list
description: >
  List installed Claude Code skills across local, workspace, and global scopes.
  Use when a user wants to see, list, show, inventory, or audit installed skills, or asks
  what skills are installed where. Triggers for: "list skills", "show installed skills",
  "what skills do I have", "inventory skills", "list global skills", "skills in my project".
---

# Skill List

Inventories installed skills in any combination of scopes. See `.claude/skills/_shared/CONVENTIONS.md` for scope paths and the manifest format.

## Step 1 — Determine scopes to scan

Default: scan `local` (cwd) and `global` (`~/.claude/skills/`).

If the user names a workspace path, also scan it. If the user names a single scope ("list global skills"), scan only that one.

## Step 2 — Discover skills per scope

For each scope, Glob `<skills_path>/*/SKILL.md`. For each match:
- Read the SKILL.md frontmatter to capture `name` and `description`.
- Read `.skill-manifest.json` in the same folder if present and capture the `origin` for the source column.
- Skills without a manifest are still listed; show source as `(unknown)`.

If a scope's skills directory does not exist, treat it as empty (do not error).

## Step 3 — Report

Print one section per scope. Use markdown tables. If a scope has no skills, say `(none)`.

```
Global (~/.claude/skills):
| name | source | description |
|---|---|---|
| dotnet-project-creator | skills/dotnet-project-creator | Scaffold .NET solutions… |

Local (./.claude/skills):
(none)
```

Truncate long descriptions to one line. Sort skills alphabetically by name within each scope.
