---
name: skill-runner
description: >
  Run a skill from this repo's local feed directly without installing it.
  Use when a user wants to try, test, execute, preview, or run a skill locally before installing.
  Triggers for: "run skill", "try skill", "test skill", "execute skill", "try out a skill",
  "preview a skill".
---

# Skill Runner

Loads a skill from `skills/<skill-name>/SKILL.md` and executes its instructions inline.

## Step 1 — Extract intent

Identify the skill name and any arguments from the user's request.

## Step 2 — Pick the skill

If not named: Glob `skills/*/SKILL.md`, Read each frontmatter, show a table of `name` / `description`, ask which to run.

## Step 3 — Load and execute

Read `skills/<skill-name>/SKILL.md` in full. Treat its body as your active instructions for this task. Pass any user-supplied arguments where the skill expects `$ARGUMENTS`.

Do not summarize or paraphrase the skill — execute it.

## Step 4 — Offer to install

After the skill completes, ask: **"Install this skill so it works outside this repo?"**

If yes, hand off to `skill-installer` for `<skill-name>`.
