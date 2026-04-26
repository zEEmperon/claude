# Claude Skill Manager

A lightweight, Claude-native skill manager for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Clone this repo, open it in Claude Code, and the meta-skills under `.claude/skills/` auto-load. Use them in plain English to list, install, remove, run, upgrade, and create skills.

No scripts to install, no daemon, no config — the skills are the manager.

## Quick start

```bash
git clone <this-repo-url> claude-skill-manager
cd claude-skill-manager
claude
```

Then talk to Claude:

> *"List my installed skills"*
>
> *"Create a skill that scaffolds a Python project"*
>
> *"Install the python-project-creator skill globally"*
>
> *"Try the python-project-creator skill"*
>
> *"Update my python-project-creator skill"*
>
> *"Remove python-project-creator from this project"*

## Meta-skills

Auto-loaded from `.claude/skills/`:

| Skill | What it does |
|---|---|
| `skill-list` | Inventory installed skills across local / workspace / global scopes. |
| `skill-installer` | Copy a skill from `skills/` into a project or the global Claude Code config. |
| `skill-remover` | Uninstall an installed skill. |
| `skill-creator` | Scaffold a new skill in `skills/<name>/`. |
| `skill-runner` | Run a skill inline from `skills/` without installing it. |
| `skill-upgrader` | Re-copy an installed skill from `skills/` to pick up local changes. |

## Concepts

- **Local feed.** `skills/<skill-name>/SKILL.md` in this repo. Gitignored. Your personal staging area: develop and try skills here, then install them where you want.
- **Install scopes.**

  | Scope | Path | Available in |
  |---|---|---|
  | `local` | `<cwd>/.claude/skills/` | Current project |
  | `workspace` | `<path>/.claude/skills/` | A specific project |
  | `global` | `~/.claude/skills/` | All Claude Code sessions |

- **Manifest.** Every installed skill carries a `.skill-manifest.json` recording its origin path and install timestamp. `skill-upgrader` reads it to know what to re-copy.

## Repo layout

```
.claude/skills/             auto-loaded meta-skills
  _shared/CONVENTIONS.md    shared reference (paths, scopes, manifest, tools)
  skill-list/
  skill-installer/
  skill-remover/
  skill-creator/
  skill-runner/
  skill-upgrader/
skills/                     gitignored — your local skill feed (flat, one folder per skill)
  <skill-name>/SKILL.md
```

## Design notes

- **Platform-agnostic.** The meta-skills use Claude's native tools (Read, Write, Glob, Edit) for file work and reach for Bash only when shell is genuinely needed. Bash via git-bash works on Linux, macOS, and Windows, so commands aren't triplicated.
- **Token-efficient.** Each meta-skill is short. The Claude Code router only loads frontmatter to decide whether a skill applies; the body is loaded only when it fires.
- **No external state.** All state lives next to the installed skill (the manifest). Nothing in the manager is per-machine.
