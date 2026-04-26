---
name: skill-creator
description: >
  Create a new skill in this repository. Use when the user wants to add, create, or write a new skill
  to this repo — regardless of category. Triggers for: "add a skill", "create a skill", "new skill",
  "write a skill for", "add skill to this repo".
---

# Skill Creator

Scaffolds a new skill in this repository and keeps `CLAUDE.md` and `README.md` in sync.

---

## Step 1 — Gather Information

Ask the user **in one message**:

1. **Category** — the category folder under `skills/`, e.g. `dotnet`, `python`, `git`. Use lowercase, hyphen-separated words.
2. **Skill name** — the skill's folder name, e.g. `project-creator`, `test-runner`. Use lowercase, hyphen-separated words. This becomes the `name` in the frontmatter.
3. **Description** — one or two sentences describing what the skill does and when it should be triggered. This becomes the `description` frontmatter field and must be keyword-rich so Claude Code can discover it automatically.
4. **What the skill should do** — enough detail to write the skill body (steps, tools used, decisions, outputs).

---

## Step 2 — Create the Skill File

Create `skills/<category>/<skill-name>/SKILL.md` with this structure:

```markdown
---
name: <skill-name>
description: >
  <description>
---

# <Title>

<Body: step-by-step procedure with all detail needed to complete the task>
```

Rules:
- The `name` field must match the folder name exactly.
- The `description` must include trigger phrases — words and phrases the user would naturally say to invoke this skill.
- The body must be self-contained: include all steps, commands, decision points, and expected outputs.
- Use `## Step N — Title` headings to structure multi-step procedures.

---

## Step 3 — Update CLAUDE.md

Add a row to the **Available Skills** table in `CLAUDE.md`:

```
| `skills/<category>/<skill-name>` | <skill-name> | <one-line description> |
```

The table is under the `## Available Skills` heading.

---

## Step 4 — Update README.md

Find the `## Available Skills` section. Under the appropriate `### <Category>` heading (create it if it doesn't exist), add a row:

```
| [<category>/<skill-name>](skills/<category>/<skill-name>/SKILL.md) | <one-line description> |
```

If a new `### <Category>` heading is needed, also add the table header:
```markdown
### <Category>

| Skill | Description |
|---|---|
| [<category>/<skill-name>](skills/<category>/<skill-name>/SKILL.md) | <one-line description> |
```

---

## Step 5 — Confirm

Report:
- File created: `skills/<category>/<skill-name>/SKILL.md`
- `CLAUDE.md` updated
- `README.md` updated
- Suggested example prompts a user could say to trigger the new skill
