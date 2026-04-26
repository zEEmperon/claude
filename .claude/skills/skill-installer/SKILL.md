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
| **Local** | `<project>/.claude/skills/<name>/` | That project only |
| **Global** | `~/.claude/skills/<name>/` | All Claude Code sessions |

> `<name>` is the value of the `name` field in the skill's YAML frontmatter — **not** the category/path used in this repo. Claude Code only discovers skills one level deep inside a skills directory.

---

## Step 1 — Detect OS

Run the following to detect the operating system:

```bash
uname -s 2>/dev/null || echo "Windows"
```

- Output starts with `Linux` → Linux
- Output starts with `Darwin` → macOS
- Output contains `MINGW` or `MSYS` → Windows via Git Bash. Use the **MINGW64** commands below.
- Output is `Windows` or command fails → Native Windows PowerShell. Use the **PowerShell** commands below.

> **MINGW64 note:** when running on Windows through Git Bash, bash expands `$variable` before PowerShell sees it. Always resolve the Windows home path via `cmd.exe` into a bash variable first, then pass that variable to PowerShell.

---

## Step 2 — Locate This Repository's Root

The root of this repository is the directory containing `CLAUDE.md` and the `skills/` folder. You already know this because Claude Code loaded this skill from it. Store it as `<REPO_ROOT>`.

---

## Step 3 — List Available Skills

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

## Step 4 — Extract Intent and Gather Only Missing Information

Before asking the user anything, read their original message and extract what you already know:

- **Skill name/path**: did they name a skill? (e.g. "install dotnet/project-creator")
- **Scope**: did they say "globally", "global", "into my project", or "locally"?

**Local always means the current working directory** — the project Claude Code is running in. Never ask the user for a path.

Ask only for what is genuinely missing, in a single message:
- If skill unknown: show the list from Step 3 and ask which to install
- If scope unknown: ask global or local
- If skill + scope are both known: proceed directly without asking anything

> Never proceed without a confirmed skill and scope.

---

## Step 5 — Resolve Target Path

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

**If local:** `<TARGET>` is the current working directory (where `claude` was executed). Resolve it:

Linux/macOS / MINGW64:
```bash
TARGET=$(pwd)
```

PowerShell (native):
```powershell
$TARGET = (Get-Location).Path
```

---

## Step 6 — Extract Skill Name and Create Destination Directory

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

## Step 7 — Copy Skill Files

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

## Step 8 — Confirm

Report to the user:

- Which skill(s) were installed
- Scope: local or global
- Destination path
- How to use it:
  - **Local**: "Open Claude Code in `<TARGET>` and use the skill. Type `/` to see it listed, or just describe the task."
  - **Global**: "The skill is now available in every Claude Code session. Type `/` to see it, or just describe the task."

---

## Installing Multiple Skills

Repeat Steps 6–8 for each selected skill before reporting in Step 9.

---

## Cross-Platform Reference

| Action | Linux/macOS | MINGW64 (Git Bash) | PowerShell (native) |
|---|---|---|---|
| Detect OS | `uname -s` → `Linux`/`Darwin` | `uname -s` → contains `MINGW` | `uname` not available |
| Get Windows home | N/A | `cmd.exe //c "echo %USERPROFILE%" \| tr -d '\r'` | `$env:USERPROFILE` |
| Create dir | `mkdir -p` | `powershell.exe -Command "New-Item -Force ..."` | `New-Item -ItemType Directory -Force` |
| Copy dir contents | `cp -r "src/." "dst/"` | `powershell.exe -Command "Copy-Item -Recurse -Force 'src\*' 'dst'"` | `Copy-Item -Recurse -Force "src\*" "dst\"` |
| Check dir exists | `test -d` | `powershell.exe -Command "Test-Path '...' -PathType Container"` | `Test-Path -PathType Container` |
