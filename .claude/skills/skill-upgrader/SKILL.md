---
name: skill-upgrader
description: >
  Update an already-installed skill to the latest version from this repository's local feed.
  Use when a user wants to update, upgrade, refresh, or sync an installed skill with the latest version.
  Only skills installed by `skill-installer` (which leaves a `.skill-manifest.json`) can be upgraded.
  Triggers for: "update skill", "upgrade skill", "refresh skill", "sync skill", "update installed skill",
  "upgrade installed skill", "get latest version of skill".
---

# Skill Upgrader

Re-copies an installed skill from this repo's local feed (`skills/<skill-name>/`) over its installed location. See `.claude/skills/_shared/CONVENTIONS.md` for scopes, manifest format, and tool-choice rules.

## Step 1 — Extract intent

Identify:
- **Skill name** (optional)
- **Scope** — `local`, `workspace`, or `global`

If scope is missing, ask. If scope is `workspace`, confirm an absolute path.

## Step 2 — List upgradeable skills

Resolve `<SKILLS_PATH>` for the scope. Glob `<SKILLS_PATH>/*/SKILL.md`. For each skill folder, Read `.skill-manifest.json` if present and capture `origin`. Skills without a manifest are not upgradeable — exclude.

If the user already named a skill, verify it has a manifest. If not, stop and report.

If no upgradeable skills are found, stop and inform the user.

## Step 3 — Resolve source

For each skill being upgraded:
- `<SOURCE>` = `<manifest.origin>` resolved against the repo root (cwd).
- Verify `<SOURCE>` exists. If the catalog entry was removed or renamed, stop and report.

`<DEST>` = `<SKILLS_PATH>/<skill-name>`.

## Step 4 — Confirm before upgrading

Show:

```
Upgrading: <skill-name>
  Source:      <SOURCE>
  Destination: <DEST>
  Action:      Overwrite all files in destination with the latest version.
Proceed? (yes/no)
```

Do not upgrade without explicit "yes".

## Step 5 — Copy

```bash
cp -r "<SOURCE>/." "<DEST>/"
```

## Step 6 — Refresh the manifest

Get the timestamp:

```bash
date -u +%Y-%m-%dT%H:%M:%SZ
```

Use Write to refresh `<DEST>/.skill-manifest.json`:

```json
{
  "name": "<skill-name>",
  "source": "local",
  "origin": "<origin>",
  "installed_at": "<timestamp>",
  "version": null
}
```

## Step 7 — Confirm

Report skill name, scope, destination, and: "If Claude Code is running, the new version takes effect immediately."

## Multiple skills

If the user asks to upgrade all upgradeable skills in a scope, repeat Steps 4–6 per skill (confirm each), then report once in Step 7.
