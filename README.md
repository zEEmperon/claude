# Claude Skills

A collection of useful [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills, organized by category and installable into any project.

## Quick Start

1. Clone this repo:
   ```bash
   git clone https://github.com/<your-username>/claude.git
   cd claude
   ```
2. Open Claude Code in this directory:
   ```bash
   claude
   ```
3. Ask Claude to install a skill:
   > *"Install [skill] globally"*
   >
   > *"Install [skill] into my current project"*
   >
   > *"Install [skill] into /path/to/any/workspace"*

The `skill-installer` skill handles the rest — it copies the skill files into the right directory.

## Meta-Skills

Meta-skills live in `.claude/skills/` and are auto-loaded when you open this repo in Claude Code. They manage the skill catalog itself.

| Skill | Trigger examples |
|---|---|
| `skill-installer` | *"Install [skill] globally"*, *"Add [skill] to my project"* |
| `skill-remover` | *"Remove [skill]"*, *"Uninstall [skill] globally"* |
| `skill-creator` | *"Create a new skill for X"*, *"Add a skill to this repo"* |

## Available Skills

### dotnet

| Skill | Description |
|---|---|
| [dotnet/project-creator](skills/dotnet/project-creator/SKILL.md) | Scaffold a .NET Core solution + project using the `dotnet` CLI. Supports C#, F#, and VB. |

## Installation Scopes

| Scope | Where skills are copied | Available in |
|---|---|---|
| **Local** | `<cwd>/.claude/skills/` | Current project only |
| **Workspace** | `<path>/.claude/skills/` | That workspace only |
| **Global** | `~/.claude/skills/` | All Claude Code sessions |

## Repo Structure

```
.claude/
  skills/
    skill-installer/   # Install skills from this repo into a project or globally
    skill-remover/     # Remove installed skills
    skill-creator/     # Scaffold a new skill in this repo
skills/
  dotnet/
    project-creator/   # Scaffold .NET solutions
```

