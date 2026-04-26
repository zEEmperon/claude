---
name: skill-upgrader
description: >
  Update an already-installed skill to the latest version from this repository's /skills catalog.
  Use when a user wants to update, upgrade, refresh, or sync an installed skill with the latest version.
  Only skills sourced from this repo's /skills catalog can be upgraded.
  Triggers for: "update skill", "upgrade skill", "refresh skill", "sync skill", "update installed skill",
  "upgrade installed skill", "get latest version of skill".
---

# Skill Upgrader

Replaces the files of an already-installed skill with the latest version from this repository's `/skills` catalog. Only skills installed by `skill-installer` (identified via a `.skill-source` marker file) can be upgraded — skills not originally installed from this catalog cannot be upgraded this way.

---

## Step 0 — Detect OS

```bash
uname -s 2>/dev/null || echo "Windows"
```

- `Linux` / `Darwin` → Linux/macOS commands
- contains `MINGW`/`MSYS` → MINGW64. Resolve home once: `WIN_HOME=$(cmd.exe //c "echo %USERPROFILE%" 2>/dev/null | tr -d '\r')`
- `Windows` or command fails → native PowerShell

> **MINGW64:** bash expands `$variable` before PowerShell sees it. Resolve the Windows home via `cmd.exe` once: `WIN_HOME=$(cmd.exe //c "echo %USERPROFILE%" 2>/dev/null | tr -d '\r')`

The repo root (`<REPO_ROOT>`) is the directory containing `CLAUDE.md` and `skills/` — already known from this skill's location.

---

## Step 1 — Extract Intent and Gather Only Missing Information

Before asking the user anything, read their original message and extract what you already know:

- **Skill name**: did they name a skill? (e.g. "update dotnet-project-creator globally")
- **Scope**: did they say "globally", "local", or provide a path to a workspace?
  - **local** = current working directory, no path needed
  - **workspace** = user provides an explicit path to another project
  - **global** = `~/.claude/`

**Local always means the current working directory.** Never ask for a path unless scope is workspace.

Ask only for what is genuinely missing, in a single message. If skill + scope are both known, proceed directly.

---

## Step 2 — List Installed Skills in Scope

List installed skills for the target scope. For each one, check for a `.skill-source` file to determine whether it is upgradeable (installed by this catalog).

**Global skills** (`~/.claude/skills/`):

Linux/macOS:
```bash
find "$HOME/.claude/skills" -maxdepth 1 -mindepth 1 -type d | sort
```

MINGW64:
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

**Workspace skills** (user-provided path):

Linux/macOS / MINGW64:
```bash
find "<PATH>/.claude/skills" -maxdepth 1 -mindepth 1 -type d | sort
```

PowerShell (native):
```powershell
Get-ChildItem -Path (Join-Path "<PATH>" ".claude\skills") -Directory | Select-Object -ExpandProperty Name | Sort-Object
```

Show the user **only the upgradeable skills** — those whose installed directory contains a `.skill-source` file written by `skill-installer`. For each installed skill:

Linux/macOS / MINGW64 — check for `.skill-source` in the installed skill dir:
```bash
cat "<SKILLS_PATH>/<SKILL_NAME>/.skill-source" 2>/dev/null
```

PowerShell (native):
```powershell
Get-Content (Join-Path "<SKILLS_PATH>" "<SKILL_NAME>" ".skill-source") -ErrorAction SilentlyContinue
```

- **`.skill-source` present**: installed by this catalog. Its content is the catalog-relative path (e.g. `skills/dotnet/project-creator`). Mark as upgradeable.
- **`.skill-source` absent**: not installed by this catalog — mark as "(not installed by this catalog — cannot upgrade)".

If the user already named a specific skill, verify it appears in the relevant scope with a `.skill-source` file. If it's not installed or lacks `.skill-source`, stop and report.

If no upgradeable skills are found in the chosen scope, stop and inform the user.

---

## Step 3 — Resolve Paths

**Source path** — read the catalog-relative path from `.skill-source` in the installed skill directory, then resolve it under `<REPO_ROOT>`:

Linux/macOS / MINGW64:
```bash
CATALOG_PATH=$(cat "<SKILLS_PATH>/<SKILL_NAME>/.skill-source")
SOURCE="<REPO_ROOT>/$CATALOG_PATH"
```

PowerShell (native):
```powershell
$catalogPath = Get-Content (Join-Path "<SKILLS_PATH>" "<SKILL_NAME>" ".skill-source")
$source = Join-Path "<REPO_ROOT>" $catalogPath
```

Verify `$SOURCE` exists in the repo. If not, the catalog entry may have been removed or renamed — stop and report.

**Destination path** — the installed skill directory:

*Global:*
- Linux/macOS: `$HOME/.claude/skills/<SKILL_NAME>`
- MINGW64: `$WIN_HOME\.claude\skills\<SKILL_NAME>`
- PowerShell: `Join-Path (Join-Path $env:USERPROFILE ".claude\skills") "<SKILL_NAME>"`

*Local:*
- Linux/macOS / MINGW64: `$(pwd)/.claude/skills/<SKILL_NAME>`
- PowerShell: `Join-Path (Join-Path (Get-Location).Path ".claude\skills") "<SKILL_NAME>"`

*Workspace:*
- Linux/macOS / MINGW64: `<PATH>/.claude/skills/<SKILL_NAME>`
- PowerShell: `Join-Path (Join-Path "<PATH>" ".claude\skills") "<SKILL_NAME>"`

Verify both `$SOURCE` and `$DEST` exist before proceeding. If either is missing, stop and report.

---

## Step 4 — Confirm Before Upgrading

Show the user what will happen and ask for confirmation:

```
Upgrading: <SKILL_NAME>
  Source:      <REPO_ROOT>/skills/<category>/<skill-path>/
  Destination: <DEST>/
  Action:      Overwrite all files in destination with latest from catalog.
Proceed? (yes/no)
```

Do **not** upgrade without explicit confirmation.

---

## Step 5 — Copy Files (Overwrite)

Copy all files from the source, overwriting the destination. The destination directory already exists (from the original install).

Linux/macOS:
```bash
cp -r "$SOURCE/." "$DEST/"
```

MINGW64 (Git Bash on Windows) — `$SOURCE` and `$DEST` are bash variables:
```bash
powershell.exe -Command "Copy-Item -Recurse -Force '$SOURCE\*' '$DEST'"
```

PowerShell (native):
```powershell
Copy-Item -Recurse -Force (Join-Path $source "*") $dest
```

---

## Step 6 — Confirm

Report to the user:

- Which skill(s) were upgraded
- Scope: local, workspace, or global
- Destination path
- Note: "The skill is now running the latest version. If Claude Code is already running, the change takes effect immediately."

Finally, print the token usage for this skill execution (input tokens, output tokens, cache reads/writes).

---

## Upgrading Multiple Skills

If the user asks to upgrade all skills in a scope (e.g. "upgrade all global skills"), repeat Steps 4–5 for each upgradeable skill (asking confirmation per skill), then report once in Step 6.
