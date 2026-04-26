---
name: skill-installer
description: >
  Install skills from this repository into a Claude Code project or globally for all projects.
  Use when a user wants to install, add, copy, or set up a skill from this repo — locally into a specific
  project or globally so it is available in every Claude Code session.
  Triggers for: "install skill", "add skill to my project", "copy skill to project", "install skill globally",
  "use skill in my project", "import skill", "install for all projects".
---

# Skill Installer

Copies a skill from this repo's local feed (`skills/<skill-name>/`) into a Claude Code skills directory. See `.claude/skills/_shared/CONVENTIONS.md` for scopes, paths, the manifest format, and tool-choice rules.

## Step 1 — Extract intent

Read the user's request. Identify:
- **Skill name** — e.g. "install dotnet-project-creator".
- **Scope** — `local`, `workspace`, or `global`. "Locally" → local. "Globally" → global. An absolute path → workspace.

## Step 2 — Fill in missing info (one message)

- If skill name is missing: Glob `skills/*/SKILL.md`, Read each frontmatter, present a table of `name` / `description`, ask which to install.
- If scope is missing: ask "global, local (this project), or a specific workspace path?".
- If scope is `workspace` and no path: ask for an absolute path.

Do not proceed without a confirmed skill and scope.

## Step 3 — Resolve paths

- `<SOURCE>` = `skills/<skill-name>/` (in the repo, i.e. cwd). Verify it exists; if not, stop.
- `<DEST_PARENT>`:
  - `local` → `.claude/skills`
  - `workspace` → `<user-path>/.claude/skills` (verify the user path exists; do not create it)
  - `global` → `~/.claude/skills` (safe to create)
- `<DEST>` = `<DEST_PARENT>/<skill-name>`

## Step 4 — Create destination and copy

```bash
mkdir -p "<DEST>"
cp -r "<SOURCE>/." "<DEST>/"
```

## Step 5 — Write the manifest

Get the timestamp:

```bash
date -u +%Y-%m-%dT%H:%M:%SZ
```

Use the Write tool to create `<DEST>/.skill-manifest.json`:

```json
{
  "name": "<skill-name>",
  "source": "local",
  "origin": "skills/<skill-name>",
  "installed_at": "<timestamp>",
  "version": null
}
```

## Step 6 — Confirm

Report:
- Skill installed
- Scope and full destination path
- How to use it:
  - `local` / `workspace`: "Open Claude Code in `<path>` to use it."
  - `global`: "Available in every Claude Code session."

## Multiple skills

Repeat Steps 3–5 per skill, then report once in Step 6.
