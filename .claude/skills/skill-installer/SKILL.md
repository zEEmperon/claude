---
name: skill-installer
description: >
  Install skills from this repository into a Claude Code project or globally for all projects.
  Use when a user wants to install, add, copy, or set up a skill from this repo — locally into a specific
  project or globally so it is available in every Claude Code session.
  Triggers for: "install skill", "add skill to my project", "copy skill to project", "install skill globally",
  "set up dotnet skill", "use skill in my project", "import skill", "install for all projects".
---

# Skill Installer

Copies a skill from this repository into a target Claude Code project (local) or into the user's global Claude Code config (global). Skills are auto-discovered by Claude Code — no registration is needed.

| Scope | Skills land in | Available |
|---|---|---|
| **Local** | `<cwd>/.claude/skills/<name>/` | Current project |
| **Workspace** | `<path>/.claude/skills/<name>/` | That workspace only |
| **Global** | `~/.claude/skills/<name>/` | All Claude Code sessions |

> Skills are installed using the `name` field from SKILL.md frontmatter as a flat folder — not the category path used in this repo.

---

## Step 1 — Detect OS and Locate Repo Root

```bash
uname -s 2>/dev/null || echo "Windows"
```

- `Linux` / `Darwin` → use Linux/macOS commands
- contains `MINGW`/`MSYS` → Windows via Git Bash (MINGW64)
- `Windows` or command fails → native PowerShell

> **MINGW64:** bash expands `$variable` before PowerShell sees it. Resolve the Windows home via `cmd.exe` once: `WIN_HOME=$(cmd.exe //c "echo %USERPROFILE%" 2>/dev/null | tr -d '\r')`

The repo root (`<REPO_ROOT>`) is the directory containing `CLAUDE.md` and `skills/` — already known from this skill's location.

---

## Step 2 — List Available Skills

Scan the `skills/` directory to discover installable skills. Each skill lives in a subdirectory containing a `SKILL.md`. Show the user the relative path and the skill's `name` from its frontmatter.

Linux/macOS:
```bash
find "<REPO_ROOT>/skills" -name "SKILL.md" | sort
```

Windows (PowerShell):
```powershell
Get-ChildItem -Path "<REPO_ROOT>\skills" -Recurse -Filter "SKILL.md" | Select-Object -ExpandProperty FullName | Sort-Object
```

Format the output for the user as a table, for example:

```
Available skills:
  dotnet/project-creator   — Create .NET Core solutions with dotnet CLI (C#, F#, VB)
```

Read each SKILL.md's `name` and `description` from its YAML frontmatter to populate this table.

---

## Step 3 — Extract Intent and Gather Only Missing Information

Before asking the user anything, read their original message and extract what you already know:

- **Skill name/path**: did they name a skill? (e.g. "install dotnet/project-creator")
- **Scope**: did they say "globally", "local", or provide a path to a workspace?
  - **local** = current working directory, no path needed
  - **workspace** = user provides an explicit path to another project
  - **global** = `~/.claude/`

Ask only for what is genuinely missing, in a single message:
- If skill unknown: show the list from Step 2 and ask which to install
- If scope unknown: ask global, local, or a specific workspace path
- If skill + scope are both known: proceed directly without asking anything

> Never proceed without a confirmed skill and scope. For workspace scope, never proceed without a confirmed absolute path.

---

## Step 4 — Resolve Target Path

**If global:** set `<TARGET>` to the user's Claude Code config directory:
- Linux/macOS: `~/.claude`
- Windows: `%USERPROFILE%\.claude`

Create it if it doesn't exist — this is safe for global scope:

Linux/macOS:
```bash
mkdir -p "$HOME/.claude"
TARGET="$HOME/.claude"
```

MINGW64 (Git Bash on Windows):
```bash
WIN_HOME=$(cmd.exe //c "echo %USERPROFILE%" 2>/dev/null | tr -d '\r')
TARGET="$WIN_HOME\.claude"
powershell.exe -Command "New-Item -ItemType Directory -Force -Path '$TARGET' | Out-Null"
```

PowerShell (native):
```powershell
$target = Join-Path $env:USERPROFILE ".claude"
New-Item -ItemType Directory -Force -Path $target | Out-Null
$TARGET = $target
```

**If local:** `<TARGET>` is the current working directory.

Linux/macOS / MINGW64:
```bash
TARGET=$(pwd)
```

PowerShell (native):
```powershell
$TARGET = (Get-Location).Path
```

**If workspace:** `<TARGET>` is the absolute path provided by the user. Verify it exists:

Linux/macOS / MINGW64:
```bash
test -d "<TARGET>" && echo "exists" || echo "missing"
```

PowerShell (native):
```powershell
Test-Path "<TARGET>" -PathType Container
```

If missing → stop and ask the user to verify the path. Do **not** create it.

---

## Step 5 — Extract Skill Name and Create Destination Directory

For each selected skill, first read the `name` field from its `SKILL.md` frontmatter:

Linux/macOS / MINGW64:
```bash
SKILL_NAME=$(grep -m1 '^name:' "<REPO_ROOT>/skills/<CATEGORY>/<SKILL_PATH>/SKILL.md" | sed 's/name:[[:space:]]*//')
```

PowerShell (native):
```powershell
$skillName = (Select-String -Path "<REPO_ROOT>\skills\<CATEGORY>\<SKILL_PATH>\SKILL.md" -Pattern '^name:\s*(.+)').Matches[0].Groups[1].Value.Trim()
```

Then create the destination using the skill name as a **flat** single-level folder:

Linux/macOS:
```bash
# Local
mkdir -p "<TARGET>/.claude/skills/$SKILL_NAME"
# Global (TARGET is already ~/.claude)
mkdir -p "<TARGET>/skills/$SKILL_NAME"
```

MINGW64 (Git Bash on Windows) — `$SKILL_NAME` and `$TARGET` are bash variables set above:
```bash
# Local
DEST="<TARGET>\.claude\skills\$SKILL_NAME"
# Global (TARGET is WIN_HOME\.claude)
DEST="$TARGET\skills\$SKILL_NAME"
powershell.exe -Command "New-Item -ItemType Directory -Force -Path '$DEST' | Out-Null"
```

PowerShell (native):
```powershell
# Local
$dest = Join-Path (Join-Path "<TARGET>" ".claude\skills") $skillName
# Global (target is already $env:USERPROFILE\.claude)
$dest = Join-Path (Join-Path "<TARGET>" "skills") $skillName
New-Item -ItemType Directory -Force -Path $dest | Out-Null
```

---

## Step 6 — Copy Skill Files

Copy all files from this repo's skill folder into the flat destination created in Step 6. **The destination directory must exist before copying.**

Linux/macOS:
```bash
# Local
cp -r "<REPO_ROOT>/skills/<CATEGORY>/<SKILL_PATH>/." "<TARGET>/.claude/skills/$SKILL_NAME/"
# Global
cp -r "<REPO_ROOT>/skills/<CATEGORY>/<SKILL_PATH>/." "<TARGET>/skills/$SKILL_NAME/"
```

MINGW64 (Git Bash on Windows) — `$DEST` is the bash variable set in Step 6:
```bash
powershell.exe -Command "Copy-Item -Recurse -Force '<REPO_ROOT>\skills\<CATEGORY>\<SKILL_PATH>\*' '$DEST'"
```

PowerShell (native) — `$dest` was set and created in Step 6:
```powershell
Copy-Item -Recurse -Force "<REPO_ROOT>\skills\<CATEGORY>\<SKILL_PATH>\*" "$dest\"
```

---

## Step 7 — Confirm

Report to the user:

- Which skill(s) were installed
- Scope: local or global
- Destination path
- How to use it:
  - **Local**: "Open Claude Code in `<TARGET>` and use the skill. Type `/` to see it listed, or just describe the task."
  - **Global**: "The skill is now available in every Claude Code session. Type `/` to see it, or just describe the task."

---

## Installing Multiple Skills

Repeat Steps 5–6 for each selected skill, then report once in Step 7.
