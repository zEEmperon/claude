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

- Output starts with `Linux` → Linux
- Output starts with `Darwin` → macOS
- Output contains `MINGW` or `MSYS` → Windows via Git Bash. Use the **MINGW64** commands below.
- Output is `Windows` or command fails → Native Windows PowerShell. Use the **PowerShell** commands below.

> **MINGW64 note:** bash expands `$variable` before PowerShell sees it. Resolve the Windows home path via `cmd.exe` into a bash variable first:
> ```bash
> WIN_HOME=$(cmd.exe //c "echo %USERPROFILE%" 2>/dev/null | tr -d '\r')
> ```

---

## Step 2 — Extract Intent from the User's Request

Before listing anything or asking questions, read the user's original message and extract what you already know:

- **Skill name**: did they name a skill? (e.g. "remove dotnet-project-creator")
- **Scope**: did they say "globally", "global", "from my project", or "locally"?

**Local always means the current working directory** — the project Claude Code is running in. Never ask the user for a path.

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
- If scope unknown: ask global or local
- If skill + scope are both known: proceed directly to Step 4 without asking anything

> Never proceed without a confirmed skill name and scope.

---

## Step 4 — Resolve Skills Path

**If global:**
- Linux/macOS: `$HOME/.claude/skills`
- MINGW64: `$WIN_HOME\.claude\skills` (bash variable)
- PowerShell (native): `Join-Path $env:USERPROFILE ".claude\skills"`

**If local** (current working directory):
- Linux/macOS / MINGW64: `$(pwd)/.claude/skills`
- PowerShell (native): `Join-Path (Get-Location).Path ".claude\skills"`

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

## Step 7 — Verify

Confirm the directory no longer exists:

Linux/macOS:
```bash
test -d "<SKILLS_PATH>/<SKILL_NAME>" && echo "still exists" || echo "removed"
```

Windows (PowerShell):
```powershell
Test-Path (Join-Path "<SKILLS_PATH>" "<SKILL_NAME>") -PathType Container
```

---

## Step 8 — Confirm

Report to the user:

- Which skill(s) were removed
- Scope: local or global
- Path that was deleted
- Note: "The skill will no longer appear in `/` or be triggered automatically. If Claude Code is running, the change takes effect immediately."

---

## Removing Multiple Skills

Repeat Steps 5–7 for each skill (asking confirmation per skill) before the final report in Step 8.

---

## Cross-Platform Reference

| Action | Linux/macOS | MINGW64 (Git Bash) | PowerShell (native) |
|---|---|---|---|
| Detect OS | `uname -s` → `Linux`/`Darwin` | `uname -s` → contains `MINGW` | `uname` not available |
| Get Windows home | N/A | `cmd.exe //c "echo %USERPROFILE%" \| tr -d '\r'` | `$env:USERPROFILE` |
| List skill dirs | `find ... -type d` | `powershell.exe -Command "Get-ChildItem -Directory ..."` | `Get-ChildItem -Directory` |
| Remove directory | `rm -rf "<path>"` | `powershell.exe -Command "Remove-Item -Recurse -Force '<path>'"` | `Remove-Item -Recurse -Force` |
| Check dir exists | `test -d "<path>"` | `powershell.exe -Command "Test-Path '<path>' -PathType Container"` | `Test-Path -PathType Container` |
