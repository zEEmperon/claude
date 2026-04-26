# Skill Manager Conventions

Shared reference for the meta-skills under `.claude/skills/`. Read this on demand from a meta-skill rather than duplicating its content.

## Repo layout

- This repo is the Claude Code skill manager. When opened in Claude Code, the meta-skills under `.claude/skills/` auto-load.
- `skills/` (gitignored) is the user's local skill feed. Each skill is a **flat** folder: `skills/<skill-name>/SKILL.md`. The folder name MUST equal the `name` in SKILL.md frontmatter.
- The current working directory is the repo root whenever a meta-skill runs (the meta-skills only load when this repo is open in Claude Code).

## Install scopes

| Scope | Skills path | Available in |
|---|---|---|
| `local` | `<cwd>/.claude/skills/` | The current project. No path needed. |
| `workspace` | `<path>/.claude/skills/` | A specific project. User must give an absolute path. |
| `global` | `~/.claude/skills/` | All Claude Code sessions. |

`~/.claude` resolves correctly under both Bash (git-bash on Windows) and PowerShell (`$env:USERPROFILE\.claude`). Pick whichever is convenient.

## Tool choice — native first, shell only when needed

| Operation | Use |
|---|---|
| List skills (find SKILL.md) | Glob `<skills_path>/*/SKILL.md` |
| Read frontmatter or manifest | Read |
| Write a small file (manifest, new SKILL.md) | Write |
| Edit one section in a file | Edit |
| Create directory tree | Bash `mkdir -p "<path>"` |
| Copy a directory tree | Bash `cp -r "<src>/." "<dest>/"` |
| Delete a directory tree | Bash `rm -rf "<path>"` |

The Bash tool runs git-bash on Windows, so Linux-style commands work cross-platform. Fall back to the PowerShell tool only if Bash fails.

Do **not** triplicate commands for Linux/macOS/MINGW/PowerShell in skill bodies. One command using the rules above is enough.

## Skill manifest

When a skill is installed, write `.skill-manifest.json` in the destination:

```json
{
  "name": "<skill-name>",
  "source": "local",
  "origin": "skills/<skill-name>",
  "installed_at": "<UTC ISO-8601>",
  "version": null
}
```

- `name` — the skill name (folder name).
- `source` — `"local"` (installed from this repo's local feed). Reserved for future source types.
- `origin` — repo-relative path of the skill in this manager.
- `installed_at` — UTC ISO-8601 timestamp. Get via `date -u +%Y-%m-%dT%H:%M:%SZ`.
- `version` — reserved; set to `null`.

## Frontmatter shape

Every skill (meta-skill or installable) has YAML frontmatter:

```yaml
---
name: <skill-name>
description: >
  One- or two-sentence trigger description, keyword-rich, naming the verbs and
  phrases a user would say to invoke this skill.
---
```

Validation rules:
- `name` must equal the leaf folder name.
- `name` must match `^[a-z0-9]+(-[a-z0-9]+)*$` (lowercase, digits, single hyphens).
- `description` should include trigger phrases — Claude Code routes on it.
