---
name: skill-runner
description: >
  Run a skill from the /skills catalog directly in this workspace without installing it.
  Use when a user wants to try, test, execute, preview, or run a skill locally.
  Triggers for: "run skill", "try skill", "test skill", "execute skill", "use the dotnet skill",
  "run dotnet/project-creator", "try out a skill".
---

# Skill Runner

Executes a skill from the `skills/` catalog directly in the current session without installing it. Useful for trying a skill before committing to installing it.

---

## Step 0 — Detect OS

```bash
uname -s 2>/dev/null || echo "Windows"
```

- `Linux` / `Darwin` → Linux/macOS commands
- contains `MINGW`/`MSYS` → MINGW64. Resolve home once if needed: `WIN_HOME=$(cmd.exe //c "echo %USERPROFILE%" 2>/dev/null | tr -d '\r')`
- `Windows` or fails → native PowerShell

---

## Step 1 — Extract Intent

Read the user's message and extract:
- **Which skill** to run (e.g. "run dotnet/project-creator")
- **Arguments** to pass to the skill, if any

If the skill is not named, list available skills (Step 2) and ask. If it is named, proceed to Step 3.

---

## Step 2 — List Available Skills

Linux/macOS:
```bash
find "<REPO_ROOT>/skills" -name "SKILL.md" | sort
```

MINGW64:
```bash
find "<REPO_ROOT>/skills" -name "SKILL.md" | sort
```

PowerShell (native):
```powershell
Get-ChildItem -Path "<REPO_ROOT>\skills" -Recurse -Filter "SKILL.md" | Select-Object -ExpandProperty FullName | Sort-Object
```

Show as a table with `name` and `description` from each SKILL.md's frontmatter.

---

## Step 3 — Load and Execute the Skill

Read the full contents of `skills/<category>/<skill-name>/SKILL.md`.

Then follow the skill's instructions exactly as if it had been installed and invoked — treating its content as your active instructions for this task. Pass any user-supplied arguments where the skill expects `$ARGUMENTS`.

Do not summarize or paraphrase the skill. Execute it.

---

## Step 4 — Offer to Install

After the skill completes, ask: **"Would you like to install this skill so it's available outside this repo?"**

If yes, invoke `skill-installer` for `<category>/<skill-name>`.

Finally, print the token usage for this skill execution (input tokens, output tokens, cache reads/writes).
