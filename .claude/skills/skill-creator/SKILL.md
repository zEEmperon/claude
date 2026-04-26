---
name: skill-creator
description: >
  Create a new skill in this repository. Use when the user wants to add, create, or write a new skill
  to this repo. Triggers for: "add a skill", "create a skill", "new skill", "write a skill for",
  "add skill to this repo".
---

# Skill Creator

Scaffolds a new skill at `skills/<skill-name>/SKILL.md` in this repo's local feed. See `.claude/skills/_shared/CONVENTIONS.md` for naming and frontmatter rules.

## Step 1 — Gather information (one message)

Ask the user:

1. **Skill name** — lowercase, hyphen-separated (e.g. `python-project-creator`, `test-runner`). Becomes the folder name AND the `name` frontmatter field.
2. **Description** — one or two sentences with trigger phrases (verbs and exact words a user would say). This is what Claude Code routes on, so be specific.
3. **What the skill should do** — enough detail to write the body: steps, tools, decisions, expected outputs.

## Step 2 — Validate the name

Reject if the name:
- contains anything other than lowercase letters, digits, and hyphens
- starts or ends with a hyphen
- already exists at `skills/<skill-name>/`

If invalid, explain why and ask for a corrected name.

## Step 3 — Create the skill file

Write `skills/<skill-name>/SKILL.md`:

```markdown
---
name: <skill-name>
description: >
  <description>
---

# <Title>

## Step 1 — <first step>

<...>
```

Authoring rules:
- `name` must equal the folder name exactly.
- `description` must include trigger phrases.
- The body is self-contained: include every step, command, decision point, and expected output. The body is what Claude follows when the skill fires.
- Prefer Claude's native tools (Read, Write, Glob, Edit) over shell commands. Use Bash only when shell is genuinely needed (e.g. running an external CLI).
- Do not triplicate commands for Linux/macOS/Windows. Bash via git-bash works on all three.

## Step 4 — Confirm and offer to install

Report:
- File created: `skills/<skill-name>/SKILL.md`
- 2–3 example user prompts that would trigger the new skill (drawn from the description's trigger phrases)

Then ask: **"Install this skill now?"**

If yes, hand off to `skill-installer` for `<skill-name>` and let it ask about scope.
