---
name: skill-remover
description: >
  Remove a previously installed skill from a Claude Code project or from the global Claude Code config.
  Use when a user wants to uninstall, remove, delete, or clean up a skill.
  Triggers for: "remove skill", "uninstall skill", "delete skill", "clean up skill".
---

# Skill Remover

Removes a skill that was previously installed by `skill-installer`, either from a specific project or from the global Claude Code config.

---

## Step 1 — Detect OS

```bash
uname -s 2>/dev/null || echo "Windows"
```

- `Linux` / `Darwin` → Linux/macOS
- contains `MINGW`/`MSYS` → MINGW64. Resolve home once: `WIN_HOME=$(cmd.exe //c "echo %USERPROFILE%" 2>/dev/null | tr -d '\r')`
- `Windows` or fails → native PowerShell

---

## Step 2 — Extract Intent from the User's Request

Before listing anything or asking questions, read the user's original message and extract what you already know:

- **Skill name**: did they name a skill? (e.g. "remove dotnet-project-creator")
- **Scope**: did they say "globally", "from my project" (local), or provide a path to a specific workspace?
  - **local** = current working directory, no path needed
  - **workspace** = user provides an explicit path
  - **global** = `~/.claude/`

**Local always means the current working directory.** Never ask for a path unless scope is workspace.

Only ask for information that is genuinely missing.

---

## Step 3 — List and Confirm

List installed skills only for the relevant scope. If scope is already known, list only that scope. If skill name is already known and matches an installed skill, skip listing.

**Global skills** (`~/.claude/skills/`):

Linux/macOS:
```bash
find "$HOME/.claude/skills" -maxdepth 1 -mindepth 1 -type d | sort
```

MINGW64 (Git Bash on Windows):
```bash
powershell.exe -Command "Get-ChildItem -Path '$WIN_HOME\.claude\skills' -Directory | Select-Object -ExpandProperty Name | Sort-Object"
```

PowerShell (native):
```powershell
Get-ChildItem -Path (Join-Path $env:USERPROFILE ".claude\skills") -Directory | Select-Object -ExpandProperty Name | Sort-Object
```

**Local skills** (`.claude/skills/` in the current project):

Linux/macOS / MINGW64:
```bash
find "$(pwd)/.claude/skills" -maxdepth 1 -mindepth 1 -type d | sort
```

PowerShell (native):
```powershell
Get-ChildItem -Path (Join-Path (Get-Location).Path ".claude\skills") -Directory | Select-Object -ExpandProperty Name | Sort-Object
```

Ask only for what is missing in a single message:
- If skill name unknown: show list and ask which to remove
- If scope unknown: ask global, local, or a specific workspace path
- If workspace scope: also ask for the absolute path if not already provided
- If skill + scope are both known: proceed directly to Step 4 without asking anything

> Never proceed without a confirmed skill name and scope. For workspace scope, never proceed without a confirmed path.

---

## Step 4 — Resolve Skills Path

**If global:**
- Linux/macOS: `$HOME/.claude/skills`
- MINGW64: `$WIN_HOME\.claude\skills`
- PowerShell (native): `Join-Path $env:USERPROFILE ".claude\skills"`

**If local** (current working directory):
- Linux/macOS / MINGW64: `$(pwd)/.claude/skills`
- PowerShell (native): `Join-Path (Get-Location).Path ".claude\skills"`

**If workspace** (user-provided path):
- Linux/macOS / MINGW64: `<PATH>/.claude/skills`
- PowerShell (native): `Join-Path "<PATH>" ".claude\skills"`

Verify the path exists before proceeding. If it doesn't, stop and report it.

---

## Step 5 — Confirm Before Deleting

Show the user the full path that will be deleted and ask for confirmation before proceeding:

```
About to delete: <SKILLS_PATH>/<SKILL_NAME>/
Proceed? (yes/no)
```

Do **not** delete without explicit confirmation.

---

## Step 6 — Remove the Skill Directory

Linux/macOS:
```bash
rm -rf "<SKILLS_PATH>/<SKILL_NAME>"
```

MINGW64 (Git Bash on Windows) — use bash variables for the path:
```bash
powershell.exe -Command "Remove-Item -Recurse -Force '<SKILLS_PATH>\<SKILL_NAME>'"
```

PowerShell (native):
```powershell
Remove-Item -Recurse -Force (Join-Path "<SKILLS_PATH>" "<SKILL_NAME>")
```

---

## Step 7 — Confirm

Report to the user:

- Which skill(s) were removed
- Scope: local or global
- Path that was deleted
- Note: "The skill will no longer appear in `/` or be triggered automatically. If Claude Code is running, the change takes effect immediately."

Finally, print the token usage for this skill execution (input tokens, output tokens, cache reads/writes).

---

## Removing Multiple Skills

Repeat Steps 5–6 for each skill (asking confirmation per skill), then report once in Step 7.
