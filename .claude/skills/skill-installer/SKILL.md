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

Copies a skill from this repository into a target Claude Code project (local) or into the user's global Claude Code config (global), and registers it via an `@`-import in the appropriate `CLAUDE.md`.

| Scope | Skills land in | Registered in | Available |
|---|---|---|---|
| **Local** | `<project>/.claude/skills/…` | `<project>/CLAUDE.md` | That project only |
| **Global** | `~/.claude/skills/…` | `~/.claude/CLAUDE.md` | All Claude Code sessions |

---

## Step 1 — Detect OS

Run the following to detect the operating system:

```bash
uname -s 2>/dev/null || echo "Windows"
```

- Output starts with `Linux` → Linux
- Output starts with `Darwin` → macOS
- Output is `Windows` or command fails → Windows (use PowerShell commands)

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

## Step 4 — Gather Information

Ask the user **in one message**:
1. **Which skill(s)** to install — use the list from Step 3. Multiple allowed.
2. **Scope** — local or global?
   - **Local**: installed into a specific project only.
   - **Global**: installed into `~/.claude/` and available in every Claude Code session.
3. **Target project root** *(only if local)* — absolute path to the root of the project. The `CLAUDE.md` there will be created if absent.

> Never proceed without a confirmed scope. Never proceed with local install without a confirmed absolute path.

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

Windows (PowerShell):
```powershell
$target = Join-Path $env:USERPROFILE ".claude"
New-Item -ItemType Directory -Force -Path $target | Out-Null
```

**If local:** `<TARGET>` is the path provided by the user. Verify it exists:

Linux/macOS:
```bash
test -d "<TARGET>" && echo "exists" || echo "missing"
```

Windows (PowerShell):
```powershell
Test-Path "<TARGET>" -PathType Container
```

If the local target is missing → stop and ask the user to verify the path. Do **not** create it automatically.

---

## Step 6 — Create Destination Directory

For each selected skill (relative path `<CATEGORY>/<SKILL_NAME>`):

Linux/macOS:
```bash
mkdir -p "<TARGET>/.claude/skills/<CATEGORY>/<SKILL_NAME>"
```

Windows (PowerShell):
```powershell
New-Item -ItemType Directory -Force -Path "<TARGET>\.claude\skills\<CATEGORY>\<SKILL_NAME>"
```

> For **global** scope, the target itself is already `~/.claude`, so the path becomes `~/.claude/skills/<CATEGORY>/<SKILL_NAME>` — do **not** add `.claude` again.

---

## Step 7 — Copy Skill Files

Copy all files from this repo's skill folder into the target. This preserves any scripts, references, or assets bundled with the skill.

Linux/macOS:
```bash
cp -r "<REPO_ROOT>/skills/<CATEGORY>/<SKILL_NAME>/." "<DEST>/skills/<CATEGORY>/<SKILL_NAME>/"
```

Windows (PowerShell):
```powershell
Copy-Item -Recurse -Force "<REPO_ROOT>\skills\<CATEGORY>\<SKILL_NAME>\*" "<DEST>\skills\<CATEGORY>\<SKILL_NAME>\"
```

Where `<DEST>` is:
- Local: `<TARGET>/.claude`
- Global: `<TARGET>` (which is already `~/.claude`)

---

## Step 8 — Register in Target CLAUDE.md

The import line to add is:
```
@.claude/skills/<CATEGORY>/<SKILL_NAME>/SKILL.md
```

The `CLAUDE.md` lives at:
- Local: `<TARGET>/CLAUDE.md`
- Global: `<TARGET>/CLAUDE.md` (i.e. `~/.claude/CLAUDE.md`)

**If the file does not exist** — create it with just the import line.  
**If it exists** — check whether the import is already present. If not, append it.

Linux/macOS:
```bash
CLAUDE_FILE="<CLAUDE_MD_PATH>"
IMPORT_LINE="@.claude/skills/<CATEGORY>/<SKILL_NAME>/SKILL.md"

if [ ! -f "$CLAUDE_FILE" ]; then
  echo "$IMPORT_LINE" > "$CLAUDE_FILE"
elif ! grep -qF "$IMPORT_LINE" "$CLAUDE_FILE"; then
  printf "\n%s\n" "$IMPORT_LINE" >> "$CLAUDE_FILE"
fi
```

Windows (PowerShell):
```powershell
$claudeFile = "<CLAUDE_MD_PATH>"
$importLine = "@.claude/skills/<CATEGORY>/<SKILL_NAME>/SKILL.md"

if (-not (Test-Path $claudeFile)) {
    Set-Content -Path $claudeFile -Value $importLine
} elseif (-not (Select-String -Path $claudeFile -SimpleMatch $importLine -Quiet)) {
    Add-Content -Path $claudeFile -Value "`n$importLine"
}
```

---

## Step 9 — Confirm

Report to the user:

- Which skill(s) were installed
- Scope: local or global
- Destination path
- What was added to `CLAUDE.md`
- How to use it:
  - **Local**: "Open Claude Code in `<TARGET>` and use the skill."
  - **Global**: "The skill is now available in every Claude Code session. Just ask Claude to use it."

---

## Installing Multiple Skills

Repeat Steps 6–8 for each selected skill before reporting in Step 9.

---

## Cross-Platform Reference

| Action | Linux/macOS | Windows (PowerShell) |
|---|---|---|
| Detect OS | `uname -s` | `uname` not available; assume Windows |
| Home dir | `$HOME` | `$env:USERPROFILE` |
| Create dir | `mkdir -p` | `New-Item -ItemType Directory -Force` |
| Copy dir contents | `cp -r "src/." "dst/"` | `Copy-Item -Recurse -Force "src\*" "dst\"` |
| Check file exists | `test -f` | `Test-Path -PathType Leaf` |
| Check dir exists | `test -d` | `Test-Path -PathType Container` |
| Append to file | `printf "\n%s\n" "$line" >> file` | `Add-Content -Value "`n$line"` |
| Search in file | `grep -qF "str" file` | `Select-String -SimpleMatch -Quiet` |
